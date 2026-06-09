# Chiến Cơ Huyền Thoại — Unity Architecture

## 1. Project Structure

```
ChiếnCơHuyềnThoại/
├── Assets/
│   ├── _Game/
│   │   ├── Scripts/
│   │   │   ├── Core/                  # Entry point, game loop
│   │   │   ├── Stats/                 # StatsComponent, damage pipeline
│   │   │   ├── Combat/                # Bullet, target, status effect
│   │   │   ├── Gacha/                 # Gacha logic, rates
│   │   │   ├── Upgrade/               # Sao, cấp, tư chất, +
│   │   │   ├── Inventory/             # Player inventory, equipment
│   │   │   ├── Entity/                # Mech, Pilot, Companion
│   │   │   ├── Manager/               # Singleton managers
│   │   │   ├── UI/                    # MVC views, controllers
│   │   │   ├── Network/               # API calls, WebSocket
│   │   │   └── Save/                  # Save/load serialization
│   │   ├── ScriptableObjects/
│   │   │   ├── MechData/
│   │   │   ├── BulletData/
│   │   │   ├── EffectData/
│   │   │   ├── EquipmentData/
│   │   │   ├── CompanionData/
│   │   │   └── GachaTable/
│   │   ├── Prefabs/
│   │   │   ├── Bullets/
│   │   │   ├── Enemies/
│   │   │   ├── Effects/
│   │   │   └── UI/
│   │   ├── Scenes/
│   │   ├── Art/
│   │   │   ├── Sprites/
│   │   │   ├── VFX/
│   │   │   └── Animations/
│   │   └── Audio/
│   └── ThirdParty/                    # DOTween, etc.
└── Packages/
```

## 2. Core Systems

### 2.1 Stats System

```csharp
// Core stat container — used by ALL entities (player, mech, enemy)
[System.Serializable]
public struct StatValue
{
    public float baseValue;
    public float bonusFlat;      // cộng thẳng
    public float bonusPercent;   // nhân %
    public float finalModifier;  // Tăng ATK cuối / Miễn ST cuối

    public float Total => (baseValue + bonusFlat) * (1 + bonusPercent);
}

public class StatsComponent : MonoBehaviour
{
    public StatValue ATK;
    public StatValue DEF;
    public StatValue HP;
    public StatValue CRIT;         // tỷ lệ chí mạng 0–1
    public StatValue DMG_CRIT;     // sát thương chí mạng 0–1
    public StatValue finalATK;     // Tăng ATK cuối
    public StatValue finalDEF;     // Miễn ST cuối

    public float CurrentHP { get; set; }

    public void ApplyEquipmentBonuses(Equipment[] equipments)
    {
        foreach (var eq in equipments)
        {
            ATK.bonusFlat += eq.ATK;
            DEF.bonusFlat += eq.DEF;
            HP.bonusFlat += eq.HP;
            CRIT.bonusFlat += eq.CRIT;
            DMG_CRIT.bonusFlat += eq.DMG_CRIT;
            if (eq.isDefense)
            {
                finalDEF.bonusFlat += eq.finalDefense;
            }
            else
            {
                finalATK.bonusFlat += eq.finalAttack;
            }
        }
    }

    public void ApplyStatusEffect(StatusEffect effect)
    {
        // gọi từ StatusEffectManager
        switch (effect.type)
        {
            case EffectType.Xuyen_Thuong:
                DEF.bonusPercent -= effect.power * effect.stacks;
                finalDEF.bonusPercent -= effect.power * effect.stacks;
                break;
            // ... mỗi effect override stats tương ứng
        }
    }
}
```

### 2.2 Damage Pipeline

```
Raw ATK → DEF reduction → Element modifier → Crit check → Final modifiers → True damage
```

```csharp
public struct DamageResult
{
    public float physicalDamage;
    public float trueDamage;
    public bool isCrit;
    public ElementType element;
}

public class DamagePipeline
{
    public static DamageResult Calculate(
        StatsComponent attacker,
        StatsComponent defender,
        ElementType bulletElement,
        float skillMultiplier = 1f)
    {
        float raw = attacker.ATK.Total * skillMultiplier;

        // 1. DEF reduction
        float def = defender.DEF.Total;
        float defReduction = def / (def + 1000); // công thức giảm ST phổ biến
        float afterDef = raw * (1 - defReduction);

        // 2. Crit check
        bool isCrit = Random.value < attacker.CRIT.Total;
        if (isCrit)
            afterDef *= (1 + attacker.DMG_CRIT.Total);

        // 3. Element modifier (base)
        float elementMod = GetElementModifier(bulletElement, defender);
        float final = afterDef * elementMod;

        // 4. Final modifiers
        float atkFinal = attacker.finalATK.Total;
        float defFinal = defender.finalDEF.Total;
        final = final * (1 + atkFinal) * (1 - defFinal);

        // 5. True damage (từ hiệu ứng Trật Tự → Quang Tinh)
        float trueDmg = 0;
        var statusEffects = defender.GetComponent<StatusEffectManager>();
        if (statusEffects != null)
        {
            if (statusEffects.HasEffect(EffectType.Quang_Tinh))
            {
                trueDmg += final * 0.3f;
                final -= trueDmg;
            }
        }

        return new DamageResult
        {
            physicalDamage = Mathf.Max(0, final),
            trueDamage = trueDmg,
            isCrit = isCrit,
            element = bulletElement
        };
    }

    private static float GetElementModifier(ElementType bullet, StatsComponent defender)
    {
        // hệ số tương khắc đơn giản, có thể mở rộng
        if (!defender.TryGetComponent(out StatusEffectManager sem))
            return 1f;

        if (bullet == ElementType.Lua && sem.HasEffect(EffectType.Bong))
            return 1.5f; // Bỏng nhận thêm ST từ Lửa
        if (sem.HasEffect(EffectType.Chay))
            return 1.3f; // Cháy nhận thêm ST từ mọi nguồn

        return 1f;
    }
}
```

### 2.3 Element & Status Effect System

```csharp
public enum ElementType
{
    Lua, Nuoc, Gio, Dat, VoTinh, AnhSang, BongToi, HonDon
}

public enum EffectType
{
    // Bậc 1 (thuần)
    Bong, Bang, XeNat, NutVo, TratTu, HonLoan,
    // Bậc 2 (thuần)
    Chay, BongLanh, NghienNat, VoNat, QuangTinh, BaoLoan,
    // Kết hợp 1+1
    SocNhiet, VuBao, DauDon, DungNham, ThanhTrung, HoaNguc,
    BaoTuyet, XoiMon, PhanQuang, VucTham, PhanQuyet, LocDen,
    PhongAn, TuTruong, NhatThuc
}

// ScriptableObject cho mỗi hiệu ứng
[CreateAssetMenu(fileName = "NewEffect", menuName = "Game/Effect")]
public class EffectDefinition : ScriptableObject
{
    public EffectType type;
    public string displayName;
    [TextArea] public string description;
    public ElementType sourceElement;
    public int maxStacks = 10;
    public float powerPerStack;  // 0.05 = 5% mỗi tầng
    public bool isUltimate;      // Bậc 2
    public GameObject vfxPrefab;
    public bool isCrossEffect;   // hiệu ứng kết hợp

    // 15 effect pairs
    public EffectType combinedFrom1;
    public EffectType combinedFrom2;
}
```

```csharp
public class StatusEffectManager : MonoBehaviour
{
    private Dictionary<EffectType, int> effectStacks = new(); // tầng hiện tại
    private Dictionary<EffectType, int> tier2Triggers = new(); // đếm Bậc 1 dùng cho Bậc 2
    private Dictionary<EffectType, EffectDefinition> definitions;

    public event System.Action<EffectType, int> OnStackChanged;
    public event System.Action<EffectType> OnUltimateTriggered;

    // MA TRẬN 6x6 — tra cứu kết hợp
    private static readonly Dictionary<(EffectType, EffectType), EffectType> CrossMatrix = new()
    {
        // Băng + Bỏng → Sốc Nhiệt
        [(EffectType.Bang, EffectType.Bong)] = EffectType.SocNhiet,
        // Xé Nát + Nứt Vỡ → Vũ Bão
        [(EffectType.XeNat, EffectType.NutVo)] = EffectType.VuBao,
        // Bỏng + Xé Nát → Đau Đớn
        [(EffectType.Bong, EffectType.XeNat)] = EffectType.DauDon,
        // Bỏng + Nứt Vỡ → Dung Nham
        [(EffectType.Bong, EffectType.NutVo)] = EffectType.DungNham,
        // Bỏng + Trật Tự → Thanh Trừng
        [(EffectType.Bong, EffectType.TratTu)] = EffectType.ThanhTrung,
        // Bỏng + Hỗn Loạn → Hỏa Ngục
        [(EffectType.Bong, EffectType.HonLoan)] = EffectType.HoaNguc,
        // Băng + Xé Nát → Bão Tuyết
        [(EffectType.Bang, EffectType.XeNat)] = EffectType.BaoTuyet,
        // Băng + Nứt Vỡ → Xói Mòn
        [(EffectType.Bang, EffectType.NutVo)] = EffectType.XoiMon,
        // Băng + Trật Tự → Phản Quang
        [(EffectType.Bang, EffectType.TratTu)] = EffectType.PhanQuang,
        // Băng + Hỗn Loạn → Vực Thẳm
        [(EffectType.Bang, EffectType.HonLoan)] = EffectType.VucTham,
        // Xé Nát + Trật Tự → Phán Quyết
        [(EffectType.XeNat, EffectType.TratTu)] = EffectType.PhanQuyet,
        // Xé Nát + Hỗn Loạn → Lốc Đen
        [(EffectType.XeNat, EffectType.HonLoan)] = EffectType.LocDen,
        // Nứt Vỡ + Trật Tự → Phong Ấn
        [(EffectType.NutVo, EffectType.TratTu)] = EffectType.PhongAn,
        // Nứt Vỡ + Hỗn Loạn → Từ Trường
        [(EffectType.NutVo, EffectType.HonLoan)] = EffectType.TuTruong,
        // Trật Tự + Hỗn Loạn → Nhật Thực
        [(EffectType.TratTu, EffectType.HonLoan)] = EffectType.NhatThuc,
    };

    public void AddStack(ElementType bulletElement, int count = 1)
    {
        if (bulletElement == ElementType.VoTinh) return; // Vô Tính không có hiệu ứng

        if (bulletElement == ElementType.HonDon)
        {
            // Hỗn Độn: kích 3 hiệu ứng ngẫu nhiên
            var randoms = GetRandomTier1Effects(3);
            foreach (var r in randoms)
                AddStackInternal(r, count);
            return;
        }

        EffectType tier1 = ElementToTier1(bulletElement);
        AddStackInternal(tier1, count);
    }

    private void AddStackInternal(EffectType tier1, int count)
    {
        // Step 1: tích tầng Bậc 1
        effectStacks.TryGetValue(tier1, out int current);
        current += count;
        effectStacks[tier1] = Mathf.Min(current, 10);

        OnStackChanged?.Invoke(tier1, effectStacks[tier1]);

        // Step 2: đạt 10 tầng → áp hiệu ứng Bậc 1
        if (current >= 10)
        {
            effectStacks[tier1] = 0; // reset
            ApplyTier1Effect(tier1);
        }

        // Step 3: kiểm tra kết hợp chéo với các hiệu ứng đang active
        CheckCrossEffects(tier1);
    }

    private void ApplyTier1Effect(EffectType tier1)
    {
        // Đếm số lần kích hoạt Bậc 1
        tier2Triggers.TryGetValue(tier1, out int triggers);
        triggers++;
        tier2Triggers[tier1] = triggers;

        // 10 lần → Hiệu ứng Bậc 2
        if (triggers >= 10)
        {
            tier2Triggers[tier1] = 0;
            EffectType tier2 = Tier1ToTier2(tier1);
            TriggerUltimateEffect(tier2);
            return;
        }

        // Áp debuff Bậc 1
        ApplyTier1Debuff(tier1);

        // Kiểm tra kết hợp chéo với CÁC hiệu ứng đang có
        foreach (var kvp in effectStacks)
        {
            if (kvp.Value > 0 && kvp.Key != tier1)
            {
                var pair = (tier1, kvp.Key);
                if (CrossMatrix.TryGetValue(pair, out EffectType cross))
                {
                    AddStackInternal(cross, 1); // tích 1 tầng cho effect kết hợp
                }
            }
        }
    }

    private void CheckCrossEffects(EffectType tier1)
    {
        // Khi một effect Bậc 1 được tích tầng, kiểm tra với tất cả Bậc 1 khác
        foreach (var kvp in effectStacks)
        {
            if (kvp.Value > 0 && kvp.Key != tier1)
            {
                var pair = (tier1, kvp.Key);
                if (CrossMatrix.TryGetValue(pair, out EffectType cross))
                {
                    AddStackInternal(cross, 1);
                }
            }
        }
    }

    private void ApplyTier1Debuff(EffectType tier1)
    {
        var stats = GetComponent<StatsComponent>();
        if (stats == null) return;

        switch (tier1)
        {
            case EffectType.Bong:
                // Bỏng: chịu thêm ST từ Lửa (handled trong DamagePipeline)
                break;
            case EffectType.Bang:
                // Băng: hạn chế di chuyển
                GetComponent<MovementController>()?.Freeze(2f);
                break;
            case EffectType.XeNat:
                // Xé Nát: giảm DEF
                stats.DEF.bonusPercent -= 0.1f;
                break;
            case EffectType.NutVo:
                // Nứt Vỡ: giảm ATK
                stats.ATK.bonusPercent -= 0.1f;
                break;
            case EffectType.TratTu:
                // Trật Tự: chuyển ST thành ST chuẩn
                break;
            case EffectType.HonLoan:
                // Hỗn Loạn: tự trừ máu theo thời gian
                StartCoroutine(HonLoanDot(3f));
                break;
        }
    }

    private void TriggerUltimateEffect(EffectType tier2)
    {
        OnUltimateTriggered?.Invoke(tier2);

        switch (tier2)
        {
            case EffectType.Chay:
                // Cháy: giảm máu tối đa + nhận thêm ST từ mọi nguồn
                break;
            case EffectType.BongLanh:
                // Bỏng Lạnh: bất động vĩnh viễn
                GetComponent<MovementController>()?.PermanentFreeze();
                break;
            case EffectType.BaoLoan:
                // Bạo Loạn: tự trừ máu = ST chuẩn theo thời gian
                StartCoroutine(BaoLoanDot(5f));
                break;
        }

        // 15 cross ultimate effects handled here
        HandleCrossUltimate(tier2);
    }

    private void HandleCrossUltimate(EffectType ultimate)
    {
        switch (ultimate)
        {
            case EffectType.SocNhiet:
                // Cái Chết Tức Thì: kết liễu < 15% máu
                var hp = GetComponent<StatsComponent>().CurrentHP;
                var maxHp = GetComponent<StatsComponent>().HP.Total;
                if (hp / maxHp < 0.15f)
                    GetComponent<StatsComponent>().CurrentHP = 0;
                break;

            case EffectType.VuBao:
                // Chịu Trận: giảm mọi chỉ số còn 50%
                var stats = GetComponent<StatsComponent>();
                stats.ATK.bonusPercent -= 0.5f;
                stats.DEF.bonusPercent -= 0.5f;
                stats.CRIT.bonusPercent -= 0.5f;
                break;

            case EffectType.DauDon:
                // Xuyên Thủng: giáp và miễn thương về 0
                GetComponent<StatsComponent>().DEF.bonusPercent = -1f;
                GetComponent<StatsComponent>().finalDEF.bonusPercent = -1f;
                break;

            // ... implement all 15 cross effects
        }
    }

    public bool HasEffect(EffectType type) =>
        effectStacks.TryGetValue(type, out int stacks) && stacks > 0;

    private static EffectType ElementToTier1(ElementType e) => e switch
    {
        ElementType.Lua => EffectType.Bong,
        ElementType.Nuoc => EffectType.Bang,
        ElementType.Gio => EffectType.XeNat,
        ElementType.Dat => EffectType.NutVo,
        ElementType.AnhSang => EffectType.TratTu,
        ElementType.BongToi => EffectType.HonLoan,
        _ => throw new System.ArgumentOutOfRangeException()
    };

    private static EffectType Tier1ToTier2(EffectType t1) => t1 switch
    {
        EffectType.Bong => EffectType.Chay,
        EffectType.Bang => EffectType.BongLanh,
        EffectType.XeNat => EffectType.NghienNat,
        EffectType.NutVo => EffectType.VoNat,
        EffectType.TratTu => EffectType.QuangTinh,
        EffectType.HonLoan => EffectType.BaoLoan,
        _ => throw new System.ArgumentOutOfRangeException()
    };

    private static readonly EffectType[] AllTier1 = {
        EffectType.Bong, EffectType.Bang, EffectType.XeNat,
        EffectType.NutVo, EffectType.TratTu, EffectType.HonLoan
    };

    private static EffectType[] GetRandomTier1Effects(int count)
    {
        var pool = new List<EffectType>(AllTier1);
        var result = new List<EffectType>();
        for (int i = 0; i < count && pool.Count > 0; i++)
        {
            int idx = Random.Range(0, pool.Count);
            result.Add(pool[idx]);
            pool.RemoveAt(idx);
        }
        return result.ToArray();
    }
}
```

### 2.4 Bullet System

```csharp
public class Bullet : MonoBehaviour
{
    public ElementType element;
    public float speed;
    public float damageMultiplier = 1f;
    public StatsComponent ownerStats;

    private void OnTriggerEnter2D(Collider2D other)
    {
        if (!other.TryGetComponent(out IDamageable target)) return;

        // 1. Tính sát thương
        var dmg = DamagePipeline.Calculate(
            ownerStats,
            target.Stats,
            element,
            damageMultiplier
        );

        // 2. Tích tầng nguyên tố
        target.StatusEffect.AddStack(element);

        // 3. Gây sát thương
        target.TakeDamage(dmg);

        // 4. Spawn VFX
        EffectSpawner.Spawn(element, transform.position);

        // 5. Pool
        BulletPool.Return(this);
    }
}

public interface IDamageable
{
    StatsComponent Stats { get; }
    StatusEffectManager StatusEffect { get; }
    void TakeDamage(DamageResult damage);
    void Heal(float amount);
}
```

### 2.5 Gacha System

```csharp
public class GachaSystem
{
    [System.Serializable]
    public class GachaResult
    {
        public MechData mech;
        public Rarity rarity;
    }

    public GachaResult Pull()
    {
        // pity counters
        pullCount++;

        // 10 → S
        if (pullCount % 10 == 0)
            return new GachaResult { mech = GetRandomMech(Rarity.S), rarity = Rarity.S };

        // 100 → SS
        if (pullCount % 100 == 0)
        {
            if (pullCount % 1000 == 0) // 1000 → SS tự chọn
                return new GachaResult { mech = GetPlayerChoiceMech(), rarity = Rarity.SS };
            return new GachaResult { mech = GetRandomMech(Rarity.SS), rarity = Rarity.SS };
        }

        // tỷ lệ thường: 70% D, 20% C, 8% B, 2% A
        return new GachaResult { mech = GetWeightedRandomMech(), rarity = Rarity.D };
    }

    public GachaResult[] Pull10() =>
        Enumerable.Range(0, 10).Select(_ => Pull()).ToArray();

    private MechData GetRandomMech(Rarity rarity) { /* query từ MechDatabase */ }
    private MechData GetPlayerChoiceMech() { /* UI cho người chơi chọn */ }
}

// Crafting: 10:1
public class CraftingSystem
{
    public MechData Merge(Rarity fromRarity, MechData[] materials)
    {
        if (materials.Length < 10) return null;
        Rarity toRarity = fromRarity + 1;
        return MechDatabase.Instance.GetRandom(toRarity);
    }

    public int Dismantle(Rarity rarity) => rarity switch
    {
        Rarity.D => 2,
        Rarity.C => 20,
        Rarity.B => 200,
        Rarity.A => 2000,
        Rarity.S => 20000,
        Rarity.SS => 200000,
        _ => 0
    };
}
```

### 2.6 Class Hierarchy

```
Entity (base)
├── StatsComponent
├── StatusEffectManager
└── IDamageable

Mech (MonoBehaviour)
├── ElementType baseElement
├── Equipment[] slots (4)
├── BulletType bulletPattern
├── PassiveSkill passive
└── ActiveSkill active

Pilot (ScriptableObject)
├── string name
├── Sprite avatar
├── string[] storyDialogue
└── PassiveSkill leaderSkill

Companion (ScriptableObject)
├── StatsComponent baseStats
├── PassiveSkill passive
├── ActiveSkill skill
├── Duyen duyenLink
└── Mastery tier (Tập sự → Bậc thầy)

Equipment (ScriptableObject)
├── SlotType (Canh, Than, Dan2, Nong)
├── bool isDefense
├── float ATK, DEF, HP, CRIT, DMG_CRIT
├── float finalAttack, finalDefense
├── PassiveEffect passiveEffect
└── ActiveSkill activeSkill
```

## 3. Data Flow

```
Player Input
    │
    ▼
MechController.Fire()
    │
    ├──► BulletPool.Spawn(element, position)
    │       │
    │       ▼
    │    Bullet.OnTriggerEnter(target)
    │       │
    │       ├──► DamagePipeline.Calculate(atk, def, element, mult)
    │       │       │
    │       │       ▼
    │       │    DamageResult
    │       │
    │       ├──► target.StatusEffect.AddStack(element)
    │       │       │
    │       │       ├──► effectStacks[effect]++
    │       │       ├──► if stacks >= 10 → ApplyTier1Effect()
    │       │       ├──► if tier1Triggers >= 10 → TriggerUltimate()
    │       │       └──► CheckCrossEffects() → cross effect stack
    │       │
    │       └──► target.TakeDamage(result)
    │
    ├──► EffectSpawner.Spawn(element, pos) // VFX
    └──► BulletPool.Return(bullet)
```

## 4. ScriptableObject Architecture

```csharp
// Base
public abstract class GameData : ScriptableObject
{
    public string id;
    public string displayName;
    [TextArea] public string description;
    public Rarity rarity;
    public Sprite icon;
}

// Mech definition
[CreateAssetMenu(fileName = "NewMech", menuName = "Game/Mech")]
public class MechData : GameData
{
    public ElementType baseElement;
    public BulletPattern bulletPattern;
    public StatsComponent baseStats;
    public MechUpgradeData upgradeCurve;
    public PassiveSkill passive;
}

// Companion
[CreateAssetMenu(fileName = "NewCompanion", menuName = "Game/Companion")]
public class CompanionData : GameData
{
    public StatsComponent baseStats;
    public PassiveSkill passive;
    public ActiveSkill active;
    public DuyenData duyenLink;
    public MasteryTier maxMastery;
}

// Equipment
[CreateAssetMenu(fileName = "NewEquipment", menuName = "Game/Equipment")]
public class EquipmentData : GameData
{
    public SlotType slot;
    public bool isDefense;
    public float ATK, DEF, HP, CRIT, DMG_CRIT;
    public float finalAttack, finalDefense;
    public PassiveEffect passive;
    public ActiveSkill active;
}

// Duyên
[CreateAssetMenu(fileName = "NewDuyen", menuName = "Game/Duyen")]
public class DuyenData : ScriptableObject
{
    public string pairId; // mech or companion id
    public string description;
    public StatValue[] bonuses; // chỉ số kích hoạt khi ghép đúng
}
```

## 5. Save/Load System

```csharp
[System.Serializable]
public class PlayerSaveData
{
    public string playerId;
    public string pilotId;
    public List<OwnedMech> mechs = new();
    public List<OwnedCompanion> companions = new();
    public List<EquipmentData> equipment = new();
    public List<Resource> resources = new();
    public int gachaPulls;
    public long lastLogin;
}

[System.Serializable]
public class OwnedMech
{
    public string mechId;
    public int stars;        // sao hiện tại
    public int level;        // cấp
    public int mastery;      // tư chất
    public int enhance;      // + mấy
    public string[] equippedItemIds;
}

public class SaveManager : MonoBehaviour
{
    private const string SAVE_KEY = "player_save";

    public void Save(PlayerSaveData data)
    {
        string json = JsonUtility.ToJson(data);
        PlayerPrefs.SetString(SAVE_KEY, json);
        PlayerPrefs.Save();

        // Sync lên server
        StartCoroutine(NetworkManager.SyncSave(data));
    }

    public PlayerSaveData Load()
    {
        string json = PlayerPrefs.GetString(SAVE_KEY, "");
        if (string.IsNullOrEmpty(json))
            return new PlayerSaveData();

        var data = JsonUtility.FromJson<PlayerSaveData>(json);

        // Verify với server
        StartCoroutine(NetworkManager.VerifySave(data));
        return data;
    }
}
```

## 6. UI Architecture (MVP)

```csharp
// Model — data
public class InventoryModel
{
    public List<OwnedMech> mechs;
    public event System.Action OnChanged;

    public void EquipMech(string mechId) { /* logic */ }
}

// View — prefab
public class InventoryView : MonoBehaviour
{
    [SerializeField] private Transform mechGrid;
    [SerializeField] private MechCard cardPrefab;

    public void Display(List<OwnedMech> mechs)
    {
        foreach (var m in mechs)
        {
            var card = Instantiate(cardPrefab, mechGrid);
            card.Setup(m);
        }
    }
}

// Presenter — glue
public class InventoryPresenter : MonoBehaviour
{
    [SerializeField] private InventoryView view;
    private InventoryModel model = new();

    private void Start()
    {
        model.OnChanged += Refresh;
        model.Load();
    }

    private void Refresh()
    {
        // clear grid
        foreach (Transform child in view.mechGrid)
            Destroy(child.gameObject);

        view.Display(model.mechs);
    }
}
```

Scene flow:
```
MainMenu → BattleSelect → GameScene → ResultScreen
          → Hangar (equip/upgrade)
          → GachaScreen
          → CompanionScreen
          → Shop
```

## 7. Performance for Mobile

```csharp
// 1. Object Pool cho đạn, VFX, enemy
public class BulletPool : MonoBehaviour
{
    private static Queue<Bullet> pool = new();
    [SerializeField] private Bullet prefab;
    [SerializeField] private int initialSize = 50;

    private void Awake()
    {
        for (int i = 0; i < initialSize; i++)
        {
            var b = Instantiate(prefab);
            b.gameObject.SetActive(false);
            pool.Enqueue(b);
        }
    }

    public static Bullet Spawn(ElementType element, Vector3 pos)
    {
        if (pool.Count == 0) return null;
        var b = pool.Dequeue();
        b.element = element;
        b.transform.position = pos;
        b.gameObject.SetActive(true);
        return b;
    }

    public static void Return(Bullet b)
    {
        b.gameObject.SetActive(false);
        pool.Enqueue(b);
    }
}

// 2. Sprite Atlas cho UI — giảm draw calls
// 3. Addressables cho asset loading
// 4. VFX pooling (ParticleSystem shared pools)
// 5. UI Canvas tối thiểu: 1 Overlay, 1 World
// 6. LOD cho enemy model
// 7. Fixed timestep: 0.02 (50fps) trên Android
// 8. Texture atlas, compression ASTC
```

## 8. Backend API Integration

```
BASE_URL = https://api.chiencohuyenthoai.com/v1

POST   /auth/register          { username, password }
POST   /auth/login             { username, password } → token
POST   /auth/refresh           { refreshToken }
GET    /player/profile         → PlayerSaveData
PUT    /player/save            { PlayerSaveData }
GET    /player/inventory       → List<OwnedMech>
POST   /gacha/pull             { count } → GachaResult[]
POST   /gacha/craft            { mechIds } → MechData
POST   /gacha/dismantle        { mechId } → Resource[]
POST   /upgrade/star           { mechId }
POST   /upgrade/level          { mechId, expItems }
POST   /upgrade/mastery        { mechId, stoneId }
POST   /upgrade/enhance        { mechId, materialIds }
GET    /leaderboard            { page } → List<LeaderboardEntry>
POST   /shop/buy               { itemId, currency }
```

Unity HTTP service:

```csharp
public class NetworkManager : MonoBehaviour
{
    private static string baseUrl = "https://api.chiencohuyenthoai.com/v1";
    private static string token;

    public static IEnumerator Post<T>(string endpoint, object body, System.Action<T> onSuccess, System.Action<string> onError)
    {
        string json = JsonUtility.ToJson(body);
        using var request = new UnityWebRequest(baseUrl + endpoint, "POST");
        request.uploadHandler = new UploadHandlerRaw(System.Text.Encoding.UTF8.GetBytes(json));
        request.downloadHandler = new DownloadHandlerBuffer();
        request.SetRequestHeader("Content-Type", "application/json");
        if (!string.IsNullOrEmpty(token))
            request.SetRequestHeader("Authorization", "Bearer " + token);

        yield return request.SendWebRequest();

        if (request.result == UnityWebRequest.Result.Success)
        {
            T result = JsonUtility.FromJson<T>(request.downloadHandler.text);
            onSuccess?.Invoke(result);
        }
        else
        {
            onError?.Invoke(request.error);
        }
    }
}
```

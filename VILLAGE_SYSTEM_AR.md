# 🏘️ شرح نظام القرية (Village System) بالعامية المصرية

## 📊 الفكرة العامة

نظام القرية ده بيتكون من **6 ملفات كود (Scripts)** بتشتغل مع بعض عشان اللاعب يقدر يبني قريته الصديقة للبيئة.

```
الفكرة ببساطة:
┌─────────────────────────────────────────────────────────┐
│   اللاعب عنده فلوس اسمها "GreenPoints" (نقاط خضراء)     │
│                         ↓                               │
│   بيضغط على مبنى من القايمة (زي البيت الشمسي)           │
│                         ↓                               │
│   لو معاه فلوس كفاية → المبنى بيظهر ويسحبه لمكانه        │
│                         ↓                               │
│   بيأكد → بيتخصم 100 نقطة ويتبني المبنى                 │
│                         ↓                               │
│   لو عايز يمسحه → بياخد نص فلوسه رجعة (50 نقطة)         │
└─────────────────────────────────────────────────────────┘
```

---

## 📁 ملفات النظام

| الملف                | بيعمل ايه؟                                         |
| -------------------- | -------------------------------------------------- |
| `BuildingData.cs`    | زي "الكارنيه" بتاع كل مبنى - فيه اسمه وصورته وسعره |
| `BuildingItem.cs`    | بيتحط على المبنى المتبني عشان يفتكر سعره           |
| `InventoryButton.cs` | الزرار اللي بتضغط عليه في القايمة                  |
| `Draggable.cs`       | بيخليك تسحب المبنى وتحطه في أي مكان                |
| `DeleteManager.cs`   | بيمسح المباني ويرجعلك نص فلوسك                     |
| `VillageManager.cs`  | **الرئيس** - بينظم كل حاجة                         |

---

## 1️⃣ ملف BuildingData.cs (بيانات المبنى)

**ده بيعمل ايه؟** ده زي "الكارت" اللي بيوصف كل نوع مبنى.

```csharp
using UnityEngine;

// ده ScriptableObject - يعني تقدر تعمل منه Assets في Unity
[CreateAssetMenu(fileName = "BD_NewBuilding", menuName = "GreenHeroes/BuildingData")]
public class BuildingData : ScriptableObject
{
    [Header("Display")]
    public string displayName;      // اسم المبنى اللي هيظهر للاعب (مثلاً "بيت شمسي")
    public Sprite icon;             // الصورة اللي هتظهر في القايمة
    public GameObject prefab;       // الشكل 3D/2D بتاع المبنى
    public Vector2 size = Vector2.one;  // حجم المبنى

    [Header("Cost")]
    public int greenPointsCost = 100;   // سعر المبنى = 100 نقطة

    // دي Function بترجع السعر كـ Text
    public string GetCostText()
    {
        return $"{greenPointsCost}";
    }

    // دي بتحسب الـ Refund (اللي هترجع لو مسحت المبنى)
    // دايماً نص السعر (50%)
    public int GetRefundAmount()
    {
        return greenPointsCost / 2;  // 100 ÷ 2 = 50 نقطة
    }
}
```

### يعني ايه ده بالبلدي؟

- ده زي **استمارة** بتملاها في Unity
- بتحط اسم المبنى، صورته، شكله، وسعره
- كل المباني سعرها 100 نقطة
- لو مسحت المبنى بتاخد 50 نقطة رجعة (نص السعر)

---

## 2️⃣ ملف BuildingItem.cs (المبنى المتبني)

**ده بيعمل ايه؟** ده بيتحط على كل مبنى اتبنى عشان يفتكر كان سعره كام

```csharp
using UnityEngine;

public class BuildingItem : MonoBehaviour
{
    [HideInInspector]
    public BuildingData data;        // بيانات المبنى (من الملف اللي فوق)

    [HideInInspector]
    public int originalCost = 100;   // السعر الأصلي اللي اللاعب دفعه

    // بيتنادى لما المبنى يتعمل أول مرة
    public void Init(BuildingData bd)
    {
        data = bd;
        originalCost = bd != null ? bd.greenPointsCost : 100;
        gameObject.name = bd.displayName + "(Placed)";  // بيسمي المبنى
    }

    // بيحسب الفلوس اللي هترجع لو مسحت المبنى
    public int GetRefundAmount()
    {
        return originalCost / 2;  // نص الفلوس
    }
}
```

### يعني ايه ده بالبلدي؟

- لما تبني مبنى، الملف ده بيفتكر انت دفعت كام
- لما تيجي تمسحه، بيديك نص الفلوس دي

---

## 3️⃣ ملف InventoryButton.cs (زرار في القايمة)

**ده بيعمل ايه؟** ده الزرار اللي بتضغط عليه عشان تختار مبنى تبنيه

```csharp
using UnityEngine;
using UnityEngine.UI;
using TMPro;

public class InventoryButton : MonoBehaviour
{
    public Image iconImage;      // صورة المبنى
    public TMP_Text label;       // اسم المبنى

    [HideInInspector] public BuildingData data;      // بيانات المبنى ده
    [HideInInspector] public VillageManager manager; // المدير الرئيسي

    // بيتنادى لما الـ VillageManager يعمل الـ Inventory
    public void Init(BuildingData bd, VillageManager vm)
    {
        data = bd;
        manager = vm;

        iconImage.sprite = bd.icon;      // بيحط الصورة
        label.text = bd.displayName;     // بيحط الاسم
    }

    // لما اللاعب يضغط على الزرار
    public void OnClickPlace()
    {
        AudioManager.Instance?.PlayClick();  // صوت ضغطة
        // بيقول للـ VillageManager يعمل المبنى ده
        manager.SpawnBuildingFromData(data, true);
    }
}
```

### يعني ايه ده بالبلدي؟

- ده الزرار اللي في القايمة الجانبية
- لما تضغط عليه، بيقول للنظام: "اللاعب عايز يبني الحاجة دي"

---

## 4️⃣ ملف Draggable.cs (السحب والإفلات)

**ده بيعمل ايه؟** ده اللي بيخليك تسحب المبنى وتحطه في أي مكان

```csharp
using System;
using UnityEngine;
using UnityEngine.EventSystems;

public class Draggable : MonoBehaviour, IPointerDownHandler, IDragHandler, IPointerUpHandler
{
    // عندنا حالتين: "بيتحط" أو "اتحط خلاص"
    public enum Mode { Placement, Placed }
    public Mode mode = Mode.Placement;

    float gridSize = 0.5f;                // المبنى بيلزق على شبكة (Grid)
    Action<GameObject> onConfirm;         // لما يأكد
    Action<GameObject> onCancel;          // لما يلغي

    [HideInInspector] public VillageManager villageManager;

    bool isDragging = false;              // هل بيسحب دلوقتي؟

    // ===== التهيئة =====
    public void Init(float grid, Action<GameObject> confirmCallback,
                     Action<GameObject> cancelCallback, VillageManager vm = null)
    {
        gridSize = Mathf.Max(0.01f, grid);
        onConfirm = confirmCallback;
        onCancel = cancelCallback;
        villageManager = vm;

        // لو مفيش Collider، بنضيف واحد عشان نقدر نضغط عليه
        if (GetComponent<Collider2D>() == null)
        {
            var bc = gameObject.AddComponent<BoxCollider2D>();
            bc.isTrigger = false;
        }
    }

    // ===== لما تضغط على المبنى =====
    public void OnPointerDown(PointerEventData eventData)
    {
        // لو المبنى اتبنى خلاص (Placed)، بنفتح شاشة المسح
        if (mode == Mode.Placed)
        {
            villageManager?.OnPlacedObjectClicked(gameObject);
            return;
        }

        // لو لسه بنحطه، نبدأ السحب
        isDragging = true;
    }

    // ===== وانت بتسحب =====
    public void OnDrag(PointerEventData eventData)
    {
        if (!isDragging || mode != Mode.Placement) return;

        // بنحول موقع الماوس لموقع في اللعبة
        Vector3 world = Camera.main.ScreenToWorldPoint(eventData.position);
        world.z = 0f;

        // بنلزق على الـ Grid (الشبكة)
        float snappedX = Mathf.Round(world.x / gridSize) * gridSize;
        float snappedY = Mathf.Round(world.y / gridSize) * gridSize;
        transform.position = new Vector3(snappedX, snappedY, transform.position.z);
    }

    // ===== لما تسيب الماوس =====
    public void OnPointerUp(PointerEventData eventData)
    {
        if (mode != Mode.Placement) return;
        isDragging = false;
        onConfirm?.Invoke(gameObject);
    }

    // لما الـ Placement يتأكد
    public void SetPlaced()
    {
        mode = Mode.Placed;
        isDragging = false;
    }
}
```

### يعني ايه ده بالبلدي؟

- لما تختار مبنى، الملف ده بيخليك تسحبه بالماوس
- المبنى بيلزق على **شبكة** (Grid) عشان يبقى منظم
- لما تسيب الماوس، بيظهر أزرار "تأكيد" و"إلغاء"
- لو المبنى **اتبنى خلاص** وضغطت عليه، بيفتحلك شاشة المسح

---

## 5️⃣ ملف DeleteManager.cs (مسح المباني)

**ده بيعمل ايه؟** ده بيمسح المباني وبيرجعلك نص فلوسك

```csharp
using UnityEngine;
using UnityEngine.UI;
using TMPro;

public class DeleteManager : MonoBehaviour
{
    [Header("UI")]
    public GameObject panelDeleteConfirm;    // شاشة التأكيد
    public TextMeshProUGUI labelName;        // اسم المبنى
    public TextMeshProUGUI labelRefund;      // الفلوس اللي هترجع
    public Button btnDelete;                 // زرار المسح
    public Button btnCancel;                 // زرار الإلغاء

    private GameObject selected;             // المبنى اللي تم اختياره

    void Start()
    {
        panelDeleteConfirm?.SetActive(false);
        btnDelete?.onClick.AddListener(DeleteSelected);
        btnCancel?.onClick.AddListener(ClearSelection);
    }

    // لما تختار مبنى عشان تمسحه
    public void Select(GameObject go)
    {
        if (go == null) { ClearSelection(); return; }

        selected = go;
        panelDeleteConfirm?.SetActive(true);

        // بنعرض اسم المبنى والفلوس اللي هترجع
        if (labelName != null)
            labelName.text = $"تمسح: {go.name}?";

        if (labelRefund != null)
        {
            var bi = go.GetComponent<BuildingItem>();
            int refund = (bi != null) ? bi.GetRefundAmount() : 50;
            labelRefund.text = $"هترجعلك: +{refund} نقطة";
        }
    }

    public void ClearSelection()
    {
        selected = null;
        panelDeleteConfirm?.SetActive(false);
    }

    // لما يأكد المسح
    public void DeleteSelected()
    {
        if (selected == null) return;

        // بنحسب الـ Refund (نص الفلوس)
        var bi = selected.GetComponent<BuildingItem>();
        int refundAmount = (bi != null) ? bi.GetRefundAmount() : 50;

        // بنرجع الفلوس للاعب
        GreenPointsManager.Instance?.AddPoints(refundAmount);

        // بنمسح المبنى
        Destroy(selected);
        selected = null;
        panelDeleteConfirm?.SetActive(false);

        // بنحفظ اللعبة
        FindFirstObjectByType<SaveManager>()?.SaveVillage();
    }
}
```

### يعني ايه ده بالبلدي؟

- لما تضغط على مبنى **مبني خلاص**، بتظهر شاشة: "عايز تمسحه؟ هترجعلك 50 نقطة"
- لو ضغطت "أيوه"، المبنى بيتمسح وبتاخد نص فلوسك
- اللعبة بتحفظ نفسها تلقائي بعد المسح

---

## 6️⃣ ملف VillageManager.cs (المدير الرئيسي)

**ده بيعمل ايه؟** ده **الدماغ** بتاع النظام كله - بينظم كل حاجة

```csharp
using System.Collections;
using UnityEngine;
using UnityEngine.UI;
using TMPro;

public class VillageManager : MonoBehaviour
{
    [Header("References")]
    public Transform placeRoot;              // الـ Parent بتاع كل المباني
    public Transform inventoryContent;       // القايمة الجانبية
    public GameObject inventoryItemPrefab;   // شكل الزرار

    [Header("Data")]
    public List<BuildingData> allBuildingData;   // كل أنواع المباني
    public List<GameObject> buildingPrefabs;      // أشكال المباني

    [Header("Placement")]
    public float gridSize = 0.5f;            // حجم الشبكة

    [Header("UI")]
    public GameObject panelConfirmPlacement; // شاشة التأكيد
    public Button confirmButton;
    public Button cancelButton;
    public GameObject messagePanel;
    public TextMeshProUGUI messageText;

    [Header("Systems")]
    public SaveManager saveManager;
    public DeleteManager deleteManager;
    public AudioManager audioManager;

    private GameObject currentObject = null;

    void Start()
    {
        panelConfirmPlacement?.SetActive(false);
        messagePanel?.SetActive(false);

        confirmButton?.onClick.AddListener(ConfirmCurrentPlacement);
        cancelButton?.onClick.AddListener(CancelCurrentPlacement);

        PopulateInventory();  // بنعمل القايمة
    }

    // بنعمل زرار لكل نوع مبنى في القايمة
    public void PopulateInventory()
    {
        // بنمسح القديم
        for (int i = inventoryContent.childCount - 1; i >= 0; i--)
            Destroy(inventoryContent.GetChild(i).gameObject);

        // بنعمل زرار لكل مبنى
        foreach (var data in allBuildingData)
        {
            var go = Instantiate(inventoryItemPrefab, inventoryContent);
            var ib = go.GetComponent<InventoryButton>();
            ib.Init(data, this);
            go.GetComponent<Button>()?.onClick.AddListener(ib.OnClickPlace);
        }
    }

    // دي الدالة الرئيسية - بتعمل المبنى
    public void SpawnBuildingFromData(BuildingData data, bool spawnAtCenter = false)
    {
        if (data == null) return;

        // ===== الخطوة 1: نشوف لو معاه فلوس =====
        if (!GreenPointsManager.Instance.CanAfford(data.greenPointsCost))
        {
            ShowMessage($"مفيش فلوس كفاية!\nمحتاج {data.greenPointsCost} نقطة", Color.red);
            return;
        }

        // ===== الخطوة 2: نجيب الـ Prefab =====
        int idx = allBuildingData.IndexOf(data);
        GameObject prefab = buildingPrefabs[idx];

        // ===== الخطوة 3: نعمل المبنى =====
        Vector3 worldPos = GetScreenCenterWorldPosition(0f);
        GameObject obj = Instantiate(prefab, placeRoot);
        obj.transform.position = worldPos;
        obj.name = data.displayName;

        // نضيف الـ Components
        obj.GetComponent<BuildingItem>()?.Init(data);

        var dr = obj.GetComponent<Draggable>();
        if (dr == null) dr = obj.AddComponent<Draggable>();
        dr.Init(gridSize, OnPlacementConfirmed, OnPlacementCanceled, this);

        currentObject = obj;
        panelConfirmPlacement?.SetActive(true);
    }

    // لما يأكد البناء
    public void OnPlacementConfirmed(GameObject placed)
    {
        if (placed == null) placed = currentObject;
        if (placed == null) return;

        var bi = placed.GetComponent<BuildingItem>();
        int cost = bi.data.greenPointsCost;

        // بنخصم الفلوس
        GreenPointsManager.Instance.TrySpend(cost);
        ShowMessage($"اتبنى {bi.data.displayName}!\n-{cost} نقطة", Color.green);

        // نخلي المبنى "متبني"
        placed.GetComponent<Draggable>()?.SetPlaced();

        panelConfirmPlacement?.SetActive(false);
        currentObject = null;

        saveManager?.SaveVillage();
    }

    // لما يلغي
    public void OnPlacementCanceled(GameObject canceled)
    {
        if (canceled == null) canceled = currentObject;
        if (canceled != null) Destroy(canceled);
        panelConfirmPlacement?.SetActive(false);
        currentObject = null;
    }

    // لما يضغط على مبنى متبني
    public void OnPlacedObjectClicked(GameObject obj)
    {
        deleteManager?.Select(obj);
    }

    // بتظهر رسالة للاعب
    public void ShowMessage(string text, Color color)
    {
        if (messageText != null)
        {
            messageText.text = text;
            messageText.color = color;
        }
        messagePanel?.SetActive(true);
        StartCoroutine(HideMessageAfterDelay());
    }

    IEnumerator HideMessageAfterDelay()
    {
        yield return new WaitForSeconds(2f);
        messagePanel?.SetActive(false);
    }
}
```

---

## 🔄 الـ Flow كله

```
1. اللاعب بيفتح اللعبة
   ↓
2. VillageManager بيعمل القايمة (PopulateInventory)
   ↓
3. اللاعب بيضغط على "بيت شمسي" في القايمة
   ↓
4. InventoryButton بتقول للـ VillageManager: "عايز يبني ده"
   ↓
5. VillageManager بيشيك: معاه 100 نقطة؟
   ↓
   ❌ لأ → رسالة "مفيش فلوس" وخلاص
   ✅ آه → كمل ↓
   ↓
6. المبنى بيظهر في نص الشاشة
   ↓
7. اللاعب بيسحبه لأي مكان (Draggable)
   ↓
8. اللاعب بيضغط "تأكيد"
   ↓
9. VillageManager بيخصم 100 نقطة ويظهر رسالة "اتبنى!"
   ↓
10. SaveManager بيحفظ اللعبة تلقائي
   ↓
--- بعدين ---
   ↓
11. اللاعب بيضغط على المبنى المتبني
   ↓
12. DeleteManager بيظهر شاشة: "تمسحه؟ هترجعلك 50 نقطة"
   ↓
13. اللاعب بيضغط "أيوه"
   ↓
14. المبنى بيتمسح + اللاعب بياخد 50 نقطة + اللعبة بتتحفظ
```

---

## 💡 حاجات مهمة تفتكرها

1. **كل المباني بـ 100 نقطة** - سعر موحد
2. **لو مسحت مبنى، بتاخد نص فلوسك (50 نقطة)**
3. **المباني بتلزق على شبكة (Grid)** - عشان تبقى منظمة
4. **اللعبة بتحفظ نفسها تلقائي** - بعد كل بناء أو مسح
5. **الـ VillageManager هو المايسترو** - كل حاجة بتعدي عليه

---

## 📂 مكان الملفات

```
Assets/Scripts/Village/
├── BuildingData.cs      ← بيانات المباني
├── BuildingItem.cs      ← المبنى المتبني
├── Draggable.cs         ← السحب والإفلات
├── DeleteManager.cs     ← مسح المباني
├── InventoryButton.cs   ← أزرار القايمة
└── VillageManager.cs    ← المدير الرئيسي
```

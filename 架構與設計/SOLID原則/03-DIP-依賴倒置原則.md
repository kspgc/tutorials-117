Dependency Inversion Principle  #依賴倒置原則

---

## 那個「介面」到底是什麼？

上一節 OCP 的解法一直出現一個詞：**介面**。

我們說 `make_order()` 應該依賴 `Notifier` 介面，而不是直接依賴 `EmailNotifier`。  
但問題是——

> **為什麼依賴介面就會比較好？「依賴」到底是什麼意思？**

這一節要回答的就是這個問題。而且答案會比你想的更根本：**它講的不是「介面很好用」，而是「誰該依賴誰」的方向問題。**


---

## 定義

DIP 有兩條，缺一不可：

> **1. 高層模組不應該依賴低層模組，兩者都應該依賴抽象。**  
> **2. 抽象不應該依賴細節，細節應該依賴抽象。**

第二條很容易被忽略，但它才是「倒置」兩個字的精髓。我們後面會回頭講。

先解釋名詞：

|名詞|意思|在這份點餐系統裡|
|---|---|---|
|**高層模組**|業務邏輯、決策、策略|`make_order()` 決定「怎麼建一張訂單」|
|**低層模組**|實作細節、技術手段|`save_data()` 決定「怎麼寫進 JSON 檔」|
|**抽象**|描述「做什麼」，不描述「怎麼做」|`OrderRepository.save(order)`|


---


## 問題長什麼樣子

回到那份點餐系統。目前的高層是這樣：

```python
def make_order():
    ...
    with data_lock:
        db["orders"].append(new_order)
        save_data()          # ← 直接呼叫低層
```

而低層是這樣：

```python
def save_data():
    with open(BACKUP_FILE, 'w', encoding='utf-8') as f:
        json.dump(db, f, ensure_ascii=False, indent=4)
```

`make_order()` 是業務邏輯，它關心的應該是「**訂單要成立**」。  
但它現在**直接知道**：訂單要存進一個叫 `BACKUP_FILE` 的檔案、用 `json.dump` 存、用 `utf-8` 編碼、`indent=4`。

這代表：

- **想測試 `make_order()`，就必須真的寫入檔案。** 沒有純記憶體模式。
- **想把存檔改成資料庫，就必須改 `make_order()`。** 業務邏輯被綁在儲存細節上。
- **想看 `make_order()` 的邏輯，就得先跳去看 `save_data()`。** 閱讀路徑被拉長。


---


## 方向錯在哪裡



![[Pasted image 20261110225807.png]]

高層箭頭指向低層。這是「**由上往下**」的依賴，也是大多數人寫 code 的預設方向。

**問題**：一旦低層換了（改成資料庫），高層就要跟著改。**變化會往上傳染。**


![[Pasted image 20261110230045.png]]

1. `make_order()` 依賴的是 **`OrderRepository` 這個抽象**，不再依賴 `save_data()`。
2. `JsonFileStorage` 也依賴同一個抽象（它實作了它）。
3. **低層的箭頭反過來了**——從「高層 → 低層」變成「低層 → 抽象 ← 高層」。


---


## 為什麼要倒過來

倒過來之後，好處有三個：

### 好處 1：可以測試了

```python
class FakeRepository:
    def __init__(self):
        self.saved = []
    def save(self, order):
        self.saved.append(order)   # 什麼都不寫檔

def test_make_order():
    repo = FakeRepository()
    make_order_with(repo, ...)     # 塞一個假的進去
    assert len(repo.saved) == 1    # 驗證邏輯，不用碰檔案
```

**業務邏輯的測試，不再需要碰檔案系統。** 測試變快、變穩、不會互相污染。


### 好處 2：可以換實作，不用改高層

|情境|要做的事|
|---|---|
|想改成資料庫|新增 `DatabaseStorage`，`make_order()` 不動|
|想改成雲端|新增 `CloudStorage`，`make_order()` 不動|
|想改成記憶體測試|新增 `FakeRepository`，`make_order()` 不動|

**高層永遠只認識 `OrderRepository`。**


### 好處 3：閱讀路徑變短

看 `make_order()` 的時候，你看到的是：
```python
repo.save(new_order)
```

你不需要知道它怎麼存、存去哪、用什麼格式。**業務邏輯就長得像業務邏輯。**


---


## 容易搞錯的地方


DIP 第二條說：

> **抽象不應該依賴細節，細節應該依賴抽象。**

這句話聽起來很抽象，但翻成大白話是：

> **介面應該由「高層」定義，不是由「低層」定義。**


### 錯誤示範

```java
// 低層寫的介面 —— 方法名和參數都帶著「JSON 檔案」這個實作細節
public interface JsonFileRepository {
    void saveToJsonFile(String path, Order order);
}

// 高層被迫用這個介面
public class OrderRepository implements JsonFileRepository {

    @Override
    public void saveToJsonFile(String path, Order order) {
        // ...
    }
}
```

**錯在哪**：這個介面叫 `JsonFileRepository`，方法叫 `saveToJsonFile`，參數還要傳 `path`。這代表——

> **高層的程式碼必須知道「東西是存到 JSON 檔案、而且需要一個路徑」。**

但高層的業務邏輯根本不該關心這件事。它只想知道「訂單存起來了沒」。

一旦介面長成這個樣子，之後想換成資料庫，**介面本身就要改**——這就是「抽象依賴了細節」。

```java
// 高層策略
public class OrderService {

    private final JsonFileStorage storage = new JsonFileStorage();  // 錯！

    public void placeOrder(Order order) {
        storage.save(order);   // 高層直接碰具體類
    }
}
```

**錯在哪**：`OrderService` 直接 `new` 了 `JsonFileStorage`。

- 想測試 `OrderService`，就必須真的寫入檔案
- 想換成資料庫，就必須改 `OrderService`
- `OrderService.java` 的檔案開頭會有一行 `import com.club.storage.JsonFileStorage;` ← **這個 import 就是違反 DIP 的證據**


## 正確示範

```java
// ============ 高層定義的介面 ============
// 這個介面只描述「高層需要什麼」，不描述「低層怎麼做」
public interface OrderRepository {
    void save(Order order);
}
```

```java
// ============ 高層策略 ============
public class OrderService {

    private final OrderRepository repo;

    // 透過建構子注入，高層不知道具體是誰
    public OrderService(OrderRepository repo) {
        this.repo = repo;
    }

    public void placeOrder(Order order) {
        // ... 業務邏輯
        repo.save(order);   // 只認識 OrderRepository，不認識 JSON
    }
}
```

```java
// ============ 高層策略 ============
public class OrderService {

    private final OrderRepository repo;

    // 透過建構子注入，高層不知道具體是誰
    public OrderService(OrderRepository repo) {
        this.repo = repo;
    }

    public void placeOrder(Order order) {
        // ... 業務邏輯
        repo.save(order);   // 只認識 OrderRepository，不認識 JSON
    }
}
```

```java
// ============ 組裝（通常在 main 或容器裡）============
public class Main {
    public static void main(String[] args) {
        OrderRepository repo = new JsonFileStorage();   // 只有這裡知道具體類
        OrderService service = new OrderService(repo);  // 高層拿到的是抽象
    }
}
```



---


## DIP 跟 OCP 的關係

用一句話分清楚：

|原則|處理的問題|
|---|---|
|**OCP**|**判斷**「這裡該不該抽象」|
|**DIP**|**描述**「抽象應該長什麼方向」|

OCP 告訴你：**當某個地方會頻繁變化時，把它抽成介面。**  
DIP 告訴你：**抽出來之後，高層要依賴那個介面，低層也要依賴那個介面。**

OCP 是「**要不要抽**」，DIP 是「**怎麼抽**」。


---


## DIP 同樣會過度設計

[[02-OCP-開放封閉原則]]

跟 OCP 一樣，DIP 也有過度使用的風險，而且更危險，因為「介面」看起來很專業。

> **不要每個函式都包一層介面。**

判斷該不該套用 DIP，跟 OCP 用同一組問題：

|問題|如果答案是…|
|---|---|
|這個低層的東西**會需要被替換**嗎？（測試、換資料庫、換服務）|是 → 值得倒置|
|這個低層的東西**只是這一支程式用一次**嗎？|是 → **不要倒置**|

如果 `save_data()` 整支程式只有一支 API 會用到，而且你完全沒有要換儲存方式、也沒有要寫測試——**那它直接呼叫就好。** 包一層 `OrderRepository` 只是多一個檔案讓人迷惑。

**DIP 的價值只在「有替換需求」的時候才成立。** 沒有替換需求，倒置只是多餘的間接層。


---

## 小結

- DIP 兩條：**高層不依賴低層，兩者都依賴抽象；抽象不依賴細節。**
- 「倒置」的意思是：**箭頭從「高 → 低」變成「低 → 抽象 ← 高」。**
- 好處：**可測試、可替換、閱讀路徑短。**
- **介面由高層定義，不是由低層定義。** 介面該長得像「高層的需要」，不是「低層的實作」。
- **沒有替換需求，就不要倒置。** DIP 跟 OCP 一樣，不是教條。



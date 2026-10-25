# 3-Modul: Diskret Matematika va Algoritmik Tahlil

Diskret matematika va algoritmlar nazariyasi kompyuter fanlarining arxitekturaviy yadrosidir. Ma'lumotlar tuzilmalari, rekursiv munosabatlar va graflarni xotirada qanday modellashtirish dasturning nafaqat asimptotik tezligini, balki apparat keshlarining (L1/L2/L3) samaradorligini ham belgilab beradi.

Ushbu modulda Master Teoremasining matematik isboti, binar daraxtlarning xotiradagi rekursiv modeli, graflar nazariyasi va Rust tilining qat'iy **Borrow-Checker** tizimida graflarni xotira xavfsiz (memory-safe) hamda keshga qulay (cache-friendly) qurish usullarini ko'rib chiqamiz.

---

## 1. To‘plamlar va Binar Daraxtlar: Rekursiv Munosabatlar va Master Theorem

Ko'plab samarali algoritmlar **"Bo'lib tashla va hukmronlik qil"** (Divide and Conquer) paradigmasiga tayanadi: masala bir xil turdagi kichikroq qism masalalarga bo'linadi, ular rekursiv yechiladi va natijalar birlashtiriladi.

### 1.1. Matematik Isbot: Master Teoremasi (Master Theorem)

Qism masalalarga bo'linuvchi rekursiv algoritmlarning vaqt murakkabligi quyidagi rekurrent munosabat bilan ifodalanadi:
$$T(n) = a \, T\left(\frac{n}{b}\right) + f(n)$$

bu yerda:
- $a \ge 1$ — har bir bosqichda hosil qilinadigan qism masalalar soni.
- $b > 1$ — har bir bosqichda masala hajmining qisqarish koeffitsiyenti.
- $f(n)$ — masalani bo'lish va qism natijalarni birlashtirish funksiyasi ($f(n) \ge 0$).

#### Rekursiya Daraxti Tahlili va Isbot:
Rekursiya daraxtining qatlamlarini (darajalarini) tahlil qilamiz:
1. **0-daraja (ildiz):** 1 ta masala, hajmi $n$, birlashtirish xarajati $f(n)$.
2. **1-daraja:** $a$ ta qism masala, har birining hajmi $\frac{n}{b}$, umumiy xarajat $a \cdot f\left(\frac{n}{b}\right)$.
3. **$j$-daraja:** $a^j$ ta qism masala, har birining hajmi $\frac{n}{b^j}$, umumiy xarajat $a^j \cdot f\left(\frac{n}{b^j}\right)$.
4. **Daraxt chuqurligi ($k$):** Masala hajmi $\frac{n}{b^k} = 1$ ga yetganda to'xtaydi:
   $$b^k = n \implies k = \log_b n$$
5. **Barglar soni (Leaves count):**
   $$L = a^k = a^{\log_b n} = n^{\log_b a}$$
   *(Chunki $a^{\log_b n} = b^{\log_b a \cdot \log_b n} = (b^{\log_b n})^{\log_b a} = n^{\log_b a}$)*

Kritik ko'rsatkich (Critical Exponent):
$$c_{\text{crit}} = \log_b a$$

#### Master Teoremasining 3 ta Holati:

1. **1-holat (Barglar hukmronligi):**
   Agar $f(n) = O(n^c)$ bo'lib, $c < \log_b a$ bo'lsa (ya'ni barglardagi ish ildizdagi ishdan ko'ra asimptotik kattaroq):
   $$T(n) = \Theta\left(n^{\log_b a}\right)$$
   *Misol:* Karatsuba ko'paytmasi ($a = 3, b = 2, f(n) = O(n)$). Bu yerda $c = 1 < \log_2 3 \approx 1.585 \implies T(n) = \Theta(n^{1.585})$.

2. **2-holat (Teng taqsimot):**
   Agar $f(n) = \Theta\left(n^{\log_b a} \log^k n\right)$ bo'lsa ($k \ge 0$):
   $$T(n) = \Theta\left(n^{\log_b a} \log^{k+1} n\right)$$
   *Misol 1:* Merge Sort ($a = 2, b = 2, f(n) = \Theta(n)$). $\log_2 2 = 1, k = 0 \implies T(n) = \Theta(n \log n)$.
   *Misol 2:* Binar qidiruv ($a = 1, b = 2, f(n) = \Theta(1)$). $\log_2 1 = 0 \implies T(n) = \Theta(\log n)$.

3. **3-holat (Ildiz hukmronligi):**
   Agar $f(n) = \Omega(n^c)$ bo'lib, $c > \log_b a$ bo'lsa va regulyarlik sharti $a f(n/b) \le d f(n)$ ($d < 1$) bajarilsa:
   $$T(n) = \Theta(f(n))$$

#### Binar Qidiruv Daraxti (BST) Chuqurligi:
$n$ ta tugunga ega binar daraxtning balandligi $h$ uchun quyidagi qat'iy munosabat o'rinli:
$$h \ge \lceil \log_2(n + 1) \rceil - 1$$
Balanslashgan daraxtda (AVL, Red-Black) barcha amallar (qidirish, qo'shish, o'chirish) $O(\log n)$ kafolatlangan vaqtda bajariladi.

### 1.2. Rust Xotira Modeli: Cheksiz O'lcham Muammosi va `Box<T>`

Rust tilida rekursiv ma'lumotlar tuzilmasini yaratishda boshlang'ich dasturchilar ko'pincha quyidagi xatoga duch kelishadi:

```rust,compile_fail
// Kompilyatsiya xatosi: error[E0072]: recursive type `Node` has infinite size
struct Node {
    value: i32,
    left: Node,
    right: Node,
}
```

#### Xatolikning ichki sababi:
Rust tili har qanday tipning o'lchamini kompilyatsiya vaqtida (compile-time) aniq bilishi shart (Stack allokatsiyasi uchun). Agar `Node` o'z ichida boshqa `Node` larni to'g'ridan-to'g'ri saqlasa:
$$\text{Size}(\text{Node}) = 4 + 2 \times \text{Size}(\text{Node}) = \infty$$
Bu cheksiz regressiyaga olib keladi.

#### Yechim: Indirection (Ko'rsatkich orqali ajratish) va `Box<T>`
Kompilyatorga o'lchamni aniq qilish uchun ko'rsatkich (pointer) qo'llaniladi:
```rust
struct Node<T> {
    value: T,
    left: Option<Box<Node<T>>>,
    right: Option<Box<Node<T>>>,
}
```
- `Box<T>` — Heap (uyum) xotiradagi allokatsiyaga ko'rsatkich bo'lib, Stackda qat'iy **8 bayt** (64-bit tizimlarda `usize`) joy oladi.
- `Option<Box<T>>` — Rustning **Null Pointer Optimization** mexanizmi tufayli `None` qiymatini xotirada $0$ (null pointer) bilan ifodalaydi va qo'shimcha tag baytini talab qilmaydi! O'lcham yana aniq 8 bayt bo'lib qoladi.

```text
Stack:                                  Heap:
+------------------------+              +------------------------+
| Node                   |              | Boxed Left Child       |
|  value: 42             |              |  value: 20             |
|  left: 0x7ffd_1000 ----+------------> |  left: None (0x0)      |
|  right: 0x7ffd_2000 ---+--+           |  right: None (0x0)     |
+------------------------+  |           +------------------------+
                            |           +------------------------+
                            +---------> | Boxed Right Child      |
                                        |  value: 60             |
                                        |  ...                   |
                                        +------------------------+
```

### 1.3. Amaliy Kod

Quyida binar qidiruv daraxti (BST) ning to'liq xavfsiz va xotirada samarali Rust implementatsiyasi keltirilgan:

```rust
pub struct BinarySearchTree<T: Ord> {
    root: Option<Box<Node<T>>>,
    size: usize,
}

struct Node<T: Ord> {
    val: T,
    left: Option<Box<Node<T>>>,
    right: Option<Box<Node<T>>>,
}

impl<T: Ord> BinarySearchTree<T> {
    pub fn new() -> Self {
        Self { root: None, size: 0 }
    }

    pub fn len(&self) -> usize {
        self.size
    }

    pub fn is_empty(&self) -> bool {
        self.size == 0
    }

    pub fn insert(&mut self, val: T) {
        if Self::insert_node(&mut self.root, val) {
            self.size += 1;
        }
    }

    fn insert_node(current: &mut Option<Box<Node<T>>>, val: T) -> bool {
        match current {
            Some(node) => {
                if val < node.val {
                    Self::insert_node(&mut node.left, val)
                } else if val > node.val {
                    Self::insert_node(&mut node.right, val)
                } else {
                    false // Qiymat allaqachon mavjud
                }
            }
            None => {
                *current = Some(Box::new(Node {
                    val,
                    left: None,
                    right: None,
                }));
                true
            }
        }
    }

    pub fn contains(&self, val: &T) -> bool {
        let mut curr = self.root.as_deref();
        while let Some(node) = curr {
            if val < &node.val {
                curr = node.left.as_deref();
            } else if val > &node.val {
                curr = node.right.as_deref();
            } else {
                return true;
            }
        }
        false
    }
}

fn main() {
    let mut bst = BinarySearchTree::new();
    assert!(bst.is_empty());

    let elements = [50, 30, 70, 20, 40, 60, 80];
    for &el in &elements {
        bst.insert(el);
    }

    assert_eq!(bst.len(), 7);
    assert!(bst.contains(&40));
    assert!(bst.contains(&80));
    assert!(!bst.contains(&99));
}
```

---

## 2. Graflar Nazariyasi: Matritsa va Ro‘yxat orqali ifodalash, DAG, Topological Sort va Rust Borrow-Checker

Graf — $G = (V, E)$ juftligi bo'lib, $V$ cho'qqilar (vertices) va $E$ qirralar (edges) to'plamidir. Amaliy tizimli dasturlashda vazifalarni rejalashtirish (task scheduling), kompilyatordagi bog'liqliklar graflari va paket menejerlari (Cargo, npm) graflar nazariyasiga va xususan **DAG** (Directed Acyclic Graph — Yo'naltirilgan Asiklik Graf) tuzilmasiga tayanadi.

### 2.1. Matritsa va Ro‘yxat orqali Ifodalash: Matematik Ta'rif va Asimptotik Tahlil

Grafni kompyuter xotirasida ifodalashning ikkita asosiy klassik usuli mavjud: **Qo'shnilik Matritsasi (Adjacency Matrix)** va **Qo'shnilik Ro'yxati (Adjacency List)**.

#### 1. Qo'shnilik Matritsasi (Adjacency Matrix)
Graf $|V| \times |V|$ o'lchamli $A$ ikkilik matritsasi bilan ifodalanadi:
$$A[u][v] = \begin{cases} 1, & \text{agar } (u, v) \in E \text{ bo'lsa} \\ 0, & \text{aks holda} \end{cases}$$

- **Xotira sarfi:** Qat'iy $\Theta(|V|^2)$. Qirralar soni $|E|$ dan qat'i nazar doim kvadratik hajm oladi.
- **Qirra mavjudligini tekshirish ($(u, v) \in E$):** $O(1)$ konstanta vaqt.
- **Cho'qqining barcha qo'shnilarini topish:** $O(|V|)$ vaqt (butun qatorni tekshirib chiqish kerak).
- **Qachon qo'llaniladi:** Zich graflar (Dense Graphs, $|E| \approx |V|^2$), barcha juftliklar orasidagi eng qisqa yo'lni topish (Floyd-Warshall algoritmi) va grafik protsessorlar (GPU) yordamidagi parallel matritsaviy hisob-kitoblarda.

#### 2. Qo'shnilik Ro'yxati (Adjacency List)
Har bir $u \in V$ cho'qqi uchun undan chiquvchi qirralar ro'yxati saqlanadi:
$$\text{Adj}[u] = \{v \in V \mid (u, v) \in E\}$$

- **Xotira sarfi:** $\Theta(|V| + |E|)$.
- **Qirra mavjudligini tekshirish ($(u, v) \in E$):** $O(\deg(u))$ vaqt.
- **Cho'qqining barcha qo'shnilarini topish:** $O(\deg(u))$ vaqt.
- **Qachon qo'llaniladi:** Siyrak graflar (Sparse Graphs, $|E| \ll |V|^2$). Real dasturlashdagi deyarli barcha tizimli masalalar (paket dependensiyalari, veb sahifalar tarmog'i, AST sintaktik daraxtlari) siyrak graflardir.

#### Asimptotik Taqqoslash Jadvali:
| Amal | Qo'shnilik Matritsasi | Qo'shnilik Ro'yxati |
| :--- | :--- | :--- |
| **Xotira murakkabligi** | $\Theta(|V|^2)$ | $\Theta(|V| + |E|)$ |
| **Qirra borligini tekshirish ($u \to v$)** | $O(1)$ | $O(\deg(u))$ |
| **Qo'shnilarni aylanib chiqish** | $O(|V|)$ | $O(\deg(u))$ |
| **Yangi qirra qo'shish** | $O(1)$ | $O(1)$ |
| **Qirrani o'chirish** | $O(1)$ | $O(\deg(u))$ |

### 2.2. Matematik Isbot: DAG va Topological Sort

#### Ta'rif:
Graf cho'qqilarining **Topologik tartibi** (Topological Sort) deb cho'qqilarning shunday chiziqli ketma-ketligi $\langle v_1, v_2, \dots, v_n \rangle$ ga aytiladiki, grafdagi har bir $(u, v) \in E$ yo'naltirilgan qirra uchun $u$ har doim $v$ dan oldin joylashadi.

#### Fundamental Teorema:
Yo'naltirilgan $G$ grafida topologik tartib mavjud bo'lishi uchun $G$ da hech qanday yo'naltirilgan tsikl bo'lmasligi (ya'ni graf DAG bo'lishi) **zarur va yetarli**.

#### Isbot:
1. **Zaruriylik ($\implies$):** Aytaylik, grafda topologik tartib mavjud, lekin unda tsikl bor: $v_1 \to v_2 \to \dots \to v_k \to v_1$.
   Topologik tartib ta'rifiga ko'ra:
   - $(v_1, v_2) \in E \implies v_1 < v_2$
   - $(v_2, v_3) \in E \implies v_2 < v_3$
   - $\dots$
   - $(v_k, v_1) \in E \implies v_k < v_1$
   Tranzitivlik xossasiga ko'ra $v_1 < v_k$. Biroq oxirgi qirra $v_k < v_1$ ni talab qiladi. Qat'iy tartib munosabatida $v_1 < v_1$ bo'lishi mumkin emas. Ziddiyat! Demak, grafda tsikl bo'lishi mumkin emas.

2. **Yetarlilik ($\impliedby$):** Chekli har qanday DAG kamida bitta kiruvchi darajasi (in-degree) nolga teng bo'lgan cho'qqiga ega ($\deg^-(v) = 0$).
   *Lemma:* Agar har bir cho'qqining kiruvchi darajasi $\ge 1$ bo'lsa, qirralar bo'ylab orqaga cheksiz harakatlanish mumkin. Cho'qqilar soni chekli ($|V| = n$) bo'lgani sababli, Dirixle (Kabutar) prinsipi bo'yicha kamida bitta cho'qqiga qayta tashrif buyuriladi, bu esa tsikl mavjudligini bildiradi. Tsiklsizlik shartiga ko'ra, kamida bitta $\deg^-(v) = 0$ cho'qqi mavjud bo'lishi shart.

#### Kahn Algoritmining Matematik Invarianti:
1. $\deg^-(u) = 0$ bo'lgan cho'qqi olinadi va topologik ro'yxatga qo'shiladi.
2. Ushbu cho'qqi va undan chiquvchi qirralar grafdan o'chiriladi.
3. Hosil bo'lgan qism graf yana DAG bo'lib qoladi.
4. Bu jarayon barcha cho'qqilar tugaguncha takrorlanadi.
Agar barcha $|V|$ ta cho'qqi ro'yxatga kirmasdan jarayon to'xtab qolsa, qolgan qism grafda tsikl mavjud!

### 2.3. Rust Xotira Modeli: Nega Ko'rsatkichlar Graflarda Xavfli?

C++ yoki Java dasturchilari graflarni ko'pincha tugunlarning o'zaro ko'rsatkichlari bilan qurishga o'rgangan:
```text
Tugun A  <=======>  Tugun B
```
Rust tilida buni `Rc<RefCell<Node>>` bilan amalga oshirishga urinish falokatga olib keladi:
1. **Reference Cycles & Memory Leaks:** Agar grafda ikki tomonlama yoki aylanma bog'liqlik bo'lsa, `Rc` hisoblagichi (reference count) hech qachon nolga tushmaydi. Natijada xotira tozalanmay oqib ketadi (Memory Leak).
2. **Borrow-Checker Ziddiyati:** Rustning *"Yoki bitta o'zgaruvchan (mutable), yoki ko'plab o'zgarmas (immutable) havola"* qoidasi murakkab to'rli graflarda runtime check (`RefCell`) va dinamik vahimalarga (`borrow_mut() panics`) sabab bo'ladi.
3. **Kesh Fragmentatsiyasi:** Har bir tugun heapning turli burchaklarida allokatsiya qilinadi, bu esa protsessor keshining yuzlab takt kesh-misslarga uchrashiga sabab bo'ladi.

#### Idiomatik va Samarali Yechim: **Arena Allocation / Index-Based Graph**
Cho'qqilar bitta yaxlit `Vec<Node>` da ketma-ket saqlanadi. Qirralar esa ko'rsatkichlar emas, balki `usize` massiv indekslari bilan ifodalanadi:

```text
RAM Xotira: Ketma-ket tekis massiv (Cache Locality a'lo darajada)
[ Cho'qqi 0 ] [ Cho'qqi 1 ] [ Cho'qqi 2 ] ... [ Cho'qqi N ]
  qo'shnilar:   qo'shnilar:   qo'shnilar:
  [1, 2]        [2]           []
```

Afzalliklari:
- 100% xotira xavfsiz (hech qanday `unsafe`, `Rc` yoki `RefCell` kerak emas).
- Borrow-checker bilan zarracha muammo tug'dirmaydi.
- Graf o'chirilganda butun graf bitta yo'la xotiradan tozalanadi ($O(1)$ deallocation).

### 2.4. Amaliy Kod

Quyida ikkala ifodalash shakli — **AdjacencyMatrix** va indekslarga asoslangan **AdjacencyList (DiGraph)** hamda Kahn algoritmi orqali topologik tartiblash va tsikllarni aniqlovchi to'liq Rust kodi keltirilgan:

```rust
use std::collections::VecDeque;

/// 1. Qo'shnilik Matritsasi orqali ifodalash (Adjacency Matrix)
/// Qirra mavjudligini O(1) da tekshirish uchun ideal.
pub struct GraphMatrix {
    nodes_count: usize,
    matrix: Vec<bool>, // Qatorlar bo'yicha tekis (flat) 1D xotira: i * n + j
}

impl GraphMatrix {
    pub fn new(nodes_count: usize) -> Self {
        Self {
            nodes_count,
            matrix: vec![false; nodes_count * nodes_count],
        }
    }

    pub fn add_edge(&mut self, from: usize, to: usize) {
        assert!(from < self.nodes_count && to < self.nodes_count);
        self.matrix[from * self.nodes_count + to] = true;
    }

    pub fn has_edge(&self, from: usize, to: usize) -> bool {
        if from >= self.nodes_count || to >= self.nodes_count {
            return false;
        }
        self.matrix[from * self.nodes_count + to]
    }
}

/// 2. Qo'shnilik Ro'yxati orqali ifodalash (Adjacency List)
/// Siyrak graflar va Topologik saralash uchun xotirada samarali.
#[derive(Debug)]
pub struct DiGraph {
    nodes_count: usize,
    adjacency: Vec<Vec<usize>>,
    in_degrees: Vec<usize>,
}

impl DiGraph {
    pub fn new(nodes_count: usize) -> Self {
        Self {
            nodes_count,
            adjacency: vec![Vec::new(); nodes_count],
            in_degrees: vec![0; nodes_count],
        }
    }

    pub fn add_edge(&mut self, from: usize, to: usize) {
        assert!(from < self.nodes_count && to < self.nodes_count);
        self.adjacency[from].push(to);
        self.in_degrees[to] += 1;
    }

    /// Kahn algoritmi orqali Topologik tartiblash.
    /// Tsikl aniqlansa Err(cycle_detected) qaytaradi.
    pub fn topological_sort(&self) -> Result<Vec<usize>, &'static str> {
        let mut in_degree = self.in_degrees.clone();
        let mut queue = VecDeque::new();
        let mut order = Vec::with_capacity(self.nodes_count);

        // Kirish darajasi 0 bo'lgan barcha cho'qqilarni navbatga qo'shamiz
        for (node, &deg) in in_degree.iter().enumerate() {
            if deg == 0 {
                queue.push_back(node);
            }
        }

        while let Some(u) = queue.pop_front() {
            order.push(u);

            for &v in &self.adjacency[u] {
                in_degree[v] -= 1;
                if in_degree[v] == 0 {
                    queue.push_back(v);
                }
            }
        }

        // Agar barcha cho'qqilar ro'yxatga kirmagan bo'lsa, tsikl mavjud!
        if order.len() == self.nodes_count {
            Ok(order)
        } else {
            Err("Grafda yo'naltirilgan tsikl aniqlandi!")
        }
    }
}

fn main() {
    // A. Qo'shnilik Matritsasini tekshirish
    let mut matrix_g = GraphMatrix::new(4);
    matrix_g.add_edge(0, 1);
    matrix_g.add_edge(1, 2);
    assert!(matrix_g.has_edge(0, 1));
    assert!(matrix_g.has_edge(1, 2));
    assert!(!matrix_g.has_edge(0, 2)); // Bevosita qirra yo'q

    // B. Qo'shnilik Ro'yxati va Kahn algoritmi
    // 6 ta vazifadan iborat loyiha dependensiyalari grafigi:
    // 5 -> 2, 5 -> 0, 4 -> 0, 4 -> 1, 2 -> 3, 3 -> 1
    let mut g = DiGraph::new(6);
    g.add_edge(5, 2);
    g.add_edge(5, 0);
    g.add_edge(4, 0);
    g.add_edge(4, 1);
    g.add_edge(2, 3);
    g.add_edge(3, 1);

    let order = g.topological_sort().expect("DAG bo'lishi kerak edi");
    assert_eq!(order.len(), 6);

    // Qirralarning o'rinini tekshiramiz: har doim `from` indeksi `to` dan oldin kelishi shart
    let pos = |node: usize| order.iter().position(|&x| x == node).unwrap();
    assert!(pos(5) < pos(2));
    assert!(pos(2) < pos(3));
    assert!(pos(3) < pos(1));
    assert!(pos(4) < pos(0));

    // Tsiklli graf tekshiruvi: 0 -> 1 -> 2 -> 0
    let mut cyclic_g = DiGraph::new(3);
    cyclic_g.add_edge(0, 1);
    cyclic_g.add_edge(1, 2);
    cyclic_g.add_edge(2, 0);

    assert!(cyclic_g.topological_sort().is_err());
}
```

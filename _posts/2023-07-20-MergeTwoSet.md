---
title: Merge Two Set
author: php
date: 2023-07-20 7:23:00
categories: [cp, tree]
tags: [tree, dsu]
pin: true
math: true
---

## Giới thiệu
Một kỹ thuật hiệu quả trong việc xử lý các bài toán nối 2 tập hợp nhau dạng cây. Kỹ thuật này dựa trên ý tưởng **Union-by-rank** trong DSU (Disjoint-set union) nên còn được gọi là **DSU on tree** hay **small-to-large**.

## Bài toán
Cho bài toán như sau:
> Cho một cây có $n$ nút và $q$ truy vấn. Mỗi truy vấn hãy cho biết có bao nhiêu nút con trong cây con gốc $u$ có giá trị là $x$.

## Thuật toán
### Thuật toán O($nq$)

Với mỗi truy vấn, ta đơn thuần vét trong cây con gốc $u$, và đếm số lượng nút con trong cây con đó có giá trị là $x$.

```cpp
int dfs(int u, int pa, int x) {
    int cnt = val[u] == x;
    for (int v : g[u]) 
        if (v != pa) cnt += dfs(v, u, x);
    return cnt;
}
```
### Kỹ thuật Small-to-Large Merging
Xét đồ thị như sau:

![expnode](https://github.com/HPhuoc0906/HPhuoc0906.github.io/assets/120120929/074ef25f-6ad8-46ff-acc2-a57e734b7090)

Ta nhận thấy rằng nếu chỉ gộp theo cách thông thường thì trong trường hợp tệ nhất 1 nút tối đa có thể bị gộp 7 lần. Nhưng nếu mỗi lần xét 2 thành phần, ta chỉ gộp thành phần nào không lớn hơn thành phần còn lại thì khi đó một nút chỉ gộp tối đa là 3 lần. Xét trường hợp tổng quát hơn: 
- Ta có 2 thành phần có kích thước lần lượt là $x$ và $y$ (giả sử $1 \le x \le y \le n$). Nếu ta gộp thành phần $x$ vào $y$ thì số nút cần gộp là $x$ nút. Ta có $x + y \ge 2 \times x$ kích thước của thành phần mới sau mỗi lần gộp vào. Mà tối đa ta chỉ có $n$ nút nên tổng số lần di chuyển 1 nút vào 1 tập hợp khác tối đa là $log(n)$.
-  Nếu ta sử dụng một cấu trúc dữ liệu có thể lưu trữ một nút Tổng độ phức tạp là **$O(nlog(n))$**.

```cpp
void mergeTwoSet(int id1, int id2) {
    if (sto[id1].size() < sto[id2].size())
        swap(sto[id1], sto[id2]);
    for (int val : sto[id2])
        sto[id1].push_back(val);
}
```
Bài tập mẫu:
- [CSES - Distinct Colors](https://cses.fi/problemset/task/1139/)

> Làm thế nào để ta áp dụng kĩ thuật **Small-to-Large Merging** vào bài toán trên ?
{: .prompt-info }
### Thuật toán $O(nlog(n))$
Đầu tiên, ta cần phải phân tách cây ra gồm các đỉnh nhẹ (light-vertex), đỉnh nặng (heavy-vertex), cạnh nhẹ (light-edge), cạnh nặng (heavy-edge). Trong đó:

- Đỉnh nặng (Heavy-vertex): là đỉnh con có kích thước lớn nhất so với các đỉnh con còn lại của nút $u$. (gọi $c$ là đỉnh hiện tại, $b$ là đỉnh con của $u$ khác $c$, ta có $size(c) \le size(b)$ với mọi $b$).
- Đỉnh nhẹ (Light-vertex): là các đỉnh con có kích thước không lớn hơn đỉnh con nặng của nút $u$.
- Cạnh nhẹ (Light-edge): là cạnh nối từ đỉnh cha đến đỉnh con nhẹ. 
- Cạnh nặng (Heavy-edge): là cạnh nối từ đỉnh cha đến đỉnh con nặng.

Do bài toán trên không có truy vấn cập nhật *online* nên ta sẽ tận dụng ưu thế *xử lý offline* để giải quyết

Giả sử ta đang ở nút $u$, gọi $bigChild$ là nút con nặng của $u$, $v$ là đỉnh con nhẹ của $u$. Lưu trữ một mảng $cnt[]$ với ý nghĩa $cnt[x]$ là số đỉnh có giá trị là $x$ trong cây con hiện tại. Ta sẽ thuật hiện thuật toán như sau:
1. Thực hiện tính toán cho các đỉnh nhẹ trước. Sau đó tính trên đỉnh nặng.
2. Cập các nút thuộc tập cây con của những đỉnh con nhẹ vào mảng $cnt[]$.
3. Cập nhật đỉnh hiện tại vào trong kết quả.
4. Nếu đỉnh hiện tại là một đỉnh nhẹ, ta sẽ xóa các đỉnh con trong tập cây con đỉnh nhẹ ra khỏi mảng kết quả $cnt[]$.

Giải thích thuật toán:
- Ở đây cần lưu ý mảng $cnt[]$ là một mảng toàn cục. Vì nếu làm mảng cục bộ và tính toán lại $cnt[]$ thì độ phức tạp có thể lên đến $O(n^2)$.
- Tại sao lại thực hiện tính toán trên các đỉnh nhỏ trước ?. Ở bước 3, đỉnh nếu hiện tại là đỉnh nhẹ thì ta sẽ xóa đi cây con đó để tránh ảnh hưởng kết quả trên các cây con khác. Cây con đỉnh nặng được giữ lại để kế thừa lên cho nút cha. Như đã chứng minh **Small-to-Large Merging** ở trên, khi ghép $2$ tập với nhau thì ta nên ưu tiên đỉnh có kích thước cây con lớn nhất để gộp các tập khác vào.
- Do ở bước 3, ta đã xóa tập cây con của của đỉnh nhẹ ra khỏi mảng kết quả nên sau khi thực hiện tính toán xong đỉnh nặng thì ta cần phải cập nhật 
- Tổng độ thuật toán là $O(nlog(n))$. Vì nếu xem xét kỹ hơn, ta sẽ nhận thấy rằng mỗi nút chỉ xét tối đa $log(n)$ lần. Dựa theo tính chất của **HLD** (Heavy-light Decomposition), mỗi đỉnh nhẹ có số cạnh nhẹ từ đỉnh đó tới nút gốc tối đa là $log(n)$ cạnh. Do đó ở bước 3, mặc dù là xóa các cạnh trong tập cây con đỉnh nhẹ ra như thực tế trên đường đi về gốc, mỗi đỉnh ta chỉ xóa tối đa là $log(n)$ lần. (Có thể hơi mơ hồ nhưng nếu tưởng tượng ra trường hợp một cây nhị phân hoàn chỉnh - CBT thì sẽ dễ hình dung hơn)

***Code:*** (soure: [[Tutorial] Sack (dsu on tree)](https://codeforces.com/blog/entry/44351) by [Arpa](https://codeforces.com/profile/Arpa))
```cpp
void calSz(int u, int pa) {
    sta[u] = ++num; // đánh dấu thời điểm bắt đầu thăm u
    stoEuler[num] = u; // lưu lại Euler Walk khi dfs
    for (int v : g[u]) if (v != pa) {
        calSz(v, u);
        sz[u] += sz[v]; // cập nhật kích thước cây cha với cây con
    }
    ++sz[u]; // nâng thêm đỉnh hiện tại
    fin[u] = num; // thời điểm kết thúc thăm u
}

void dfs(int u, int pa, bool keep) {
    int bigChild = -1, maxS = 0;
    for (int v : g[u]) if (v != pa && sz[v] > maxS) {
        maxS = sz[v]; // cập nhật lại kích thước lớn hơn
        bigChild = v; // và đỉnh nặng mới
    }
    for (int v : g[u])
        if (v != pa && v != bigChild)
            dfs(v, u, 0); // tính toán trên đỉnh nhẹ trước

    if (bigChild != -1) dfs(bigChild, u, 1); // tính toán trên đỉnh nặng sau
    ++cntColor[val[u]]; // đưa nút hiện tại vào kết quả

    for (int v : g[u])
        if (v != pa && v != bigChild)
            for (int i = sta[v]; i <= fin[v]; ++i) // dùng tính chất của Euler Walk để lấy nhanh các nút con
                ++cntColor[val[stoEuler[i]]]; // cập nhật tập hợp các cây con đỉnh nhẹ vào kết quả
    if (!keep) // nếu nút hiện tại là một đỉnh nhẹ
        for (int i = sta[u]; i <= fin[u]; ++i)
            --cntColor[val[stoEuler[i]]]; // xóa ra khỏi mảng kết quả
}
```
Bài tập mẫu:
- [Codeforces - E. Lomsat gelral](https://codeforces.com/problemset/problem/600/E)
- [Codeforces - D. Tree Requests](https://codeforces.com/problemset/problem/570/D)

## Tổng kết
Đây là một kỹ thuật khá phổ biến trong các bài toán liên quan đến xử lý trong các bài toán xử lý truy vấn offline trên các nút con và liên quan dến gộp các tập hợp với nhau (thường thì liên quan đến DSU). Kỹ thuật giúp gộp tập $2$ hợp nhanh và hiệu quả, đỡ tốn bộ nhớ lưu và thời gian thực hiện.
## Nguồn tham khảo
- [[Tutorial] Sack (dsu on tree)](https://codeforces.com/blog/entry/44351)
- [[Explanation] dsu on trees (small to large)](https://codeforces.com/blog/entry/67696)
- [Small-To-Large Merging - USACO Guide](https://usaco.guide/plat/merging?lang=cpp)

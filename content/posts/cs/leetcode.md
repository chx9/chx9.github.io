---
title: "leetcode"
date: 2023-07-29T11:05:24+08:00
lastmod: 2023-07-29T11:05:24+08:00
author: ["chx9"]
keywords:
  -
categories: # 没有分类界面可以不填写
  -
tags: # 标签
  -
description: ""
weight:
slug: ""
draft: false # 是否为草稿
comments: true # 本页面是否显示评论
reward: false # 打赏
mermaid: true #是否开启mermaid
showToc: true # 显示目录
TocOpen: true # 自动展开目录
hidemeta: false # 是否隐藏文章的元信息，如发布日期、作者等
disableShare: true # 底部不显示分享栏
showbreadcrumbs: true #顶部显示路径
cover:
  image: "" #图片路径例如：posts/tech/123/123.png
  zoom: # 图片大小，例如填写 50% 表示原图像的一半大小
  caption: "" #图片底部描述
  alt: ""
  relative: false
---

## 141 环形链表

```
class Solution {
public:
    bool hasCycle(ListNode *head) {
        auto fast = head;
        auto slow = head;
        while(fast && fast->next){
            slow = slow->next;
            fast = fast->next->next;
            if(slow == fast) return true;
        }
        return false;
    }
};
```

## 142 环形链表

[https://leetcode.cn/problems/linked-list-cycle-ii/?favorite=2cktkvj](https://leetcode.cn/problems/linked-list-cycle-ii/?favorite=2cktkvj)
有一个链表，如果有环，返回环的入口处，没有则返回
方法一：哈希表
哈希表第一个重复的值，就是入口处
方法二：快慢指针
如果有环，快慢指针将会再某一点相遇，此时的慢指针和 head 与环入口点距离相等

```cpp
class Solution {
public:
    ListNode *detectCycle(ListNode *head) {
        if(head == nullptr) return nullptr;
        ListNode* fast = head;
        ListNode* slow = head;
        while(fast && fast->next){
            fast = fast->next->next;
            slow = slow->next;
            if(fast == slow){
                auto res = head;
                while(res!=slow){
                    res = res->next;
                    slow = slow->next;
                }
                return res;
            }
        }
        return nullptr;
    }
};

```

## 146 LRU 缓存

[https://leetcode.cn/problems/lru-cache/](https://leetcode.cn/problems/lru-cache/)

请你设计一个`Last Recent Used Cache`
使用一个双向指针（带有 head 和 tail 节点，同时节点中包含 key 和 value 值）以及一个 unordered_map 储存，实现两个函数，moveToHead()和 removeTail()

```cpp
class LRUCache {
struct ListNode{
    int key;
    int val;
    ListNode* next;
    ListNode* prev;
    ListNode(): key(0), val(0){}
    ListNode(int key, int val):key(key), val(val){}
};
private:
    int capacity;
    unordered_map<int, ListNode*> cache;
    ListNode* head;
    ListNode* tail;
public:
    LRUCache(int capacity) {
        this->capacity = capacity;
        head = new ListNode();
        tail = new ListNode();
        head->next = tail;
        tail->next = head;
        head->prev = tail;
        tail->prev = head;
    }

    int get(int key) {
        if(cache.count(key) != 0){
            moveToHead(key);
            return cache[key]->val;
        }
        return -1;
    }

    void put(int key, int value) {
        if(cache.count(key)==0){
            addToHead(key, value);
        }else{
            cache[key]->val = value;
            moveToHead(key);
        }
        if(cache.size() > this->capacity){
            removeTail();
        }
    }
    void addToHead(int key, int value){

        auto node = new ListNode(key, value);
        cache[key] = node;

        //
        auto tp = head->next;
        head->next = node;
        node->prev = head;
        node->next = tp;
        tp->prev = node;
    }
    void moveToHead(int key){
        auto node = cache[key];
        node->prev->next = node->next;
        node->next->prev = node->prev;
        auto tp = head->next;
        //
        head->next = node;
        node->prev = head;
        node->next = tp;
        tp->prev = node;
    }
    void removeTail(){
        auto last = tail->prev;
        cache.erase(last->key);
        tail->prev = last->prev;
        last->prev->next = tail;
        delete last;
    }
};

```

## 148 排序链表

[https://leetcode.cn/problems/sort-list/?favorite=2cktkvj](https://leetcode.cn/problems/sort-list/?favorite=2cktkvj)
merge_sort 排序对链表进行排序：
1、如果 merge_sort 传入的头尾链表相距一个单位，则切断链表，返回 head
2、快慢链表找到中间值
3、合并链表
注意遍历快慢链表的方法，fast != tail，然后在循环中判断是否需要 fast 进两步

```cpp
class Solution {
public:
    ListNode* merge_sort(ListNode* head, ListNode* tail){
        if(!head ){
            return head;
        }
        if(head->next == tail){
            head->next = nullptr;
            return head;
        }
        auto fast = head;
        auto slow = head;
        while(fast!=tail && fast->next!=tail){
            fast = fast->next->next;
            slow = slow->next;
        }
        return merge(merge_sort(head, slow), merge_sort(slow, tail));
    }
    ListNode* merge(ListNode* l1, ListNode*l2){
        auto dummy = new ListNode();
        auto cur = dummy;
        while(l1 && l2){
            if(l1->val < l2->val){
                cur->next = l1;
                l1 = l1->next;
            }else{
                cur->next = l2;
                l2 = l2->next;
            }
            cur = cur->next;
        }
        if(l1){
            cur->next = l1;
        }else{
            cur->next = l2;
        }
        return dummy->next;
    }
    ListNode* sortList(ListNode* head) {
        return merge_sort(head, nullptr);
    }
};
```

## 152.乘积最大子数组

[https://leetcode.cn/problems/maximum-product-subarray/](https://leetcode.cn/problems/maximum-product-subarray/)

动态规划，因为数组中可能有负数，所以不仅仅要维护一个当前最大动态数组，还要维护当前最小数组，防止负负得正变成最大值

```cpp
class Solution {
public:
    int maxProduct(vector<int>& nums) {
        if( nums.size()==0 ) return 0;
        vector<int> curMax(nums.size());
        vector<int> curMin(nums.size());
        int res = nums[0];
        curMax[0] = nums[0];
        curMin[0] = nums[0];
        for(int i=1;i<nums.size();i++){
            int cur = nums[i];
            curMax[i] = max(max(curMax[i-1]*cur, curMin[i-1]*cur), cur);
            curMin[i] = min(min(curMin[i-1]*cur, curMax[i-1]*cur), cur);
            res = max(res, curMax[i]);
        }
        return res;
    }
};

```

## 155 最小栈

[https://leetcode.cn/problems/min-stack/](https://leetcode.cn/problems/min-stack/)

维护两个栈，除了基础的栈外，维护一个储存最小值的栈

```cpp
class MinStack {
private:
    stack<int> min_st;
    stack<int> st;
public:
    MinStack() {

    }
    void push(int val) {
        st.push(val);
        if(min_st.empty() || min_st.top() >= val){
            min_st.push(val);
        }
    }
    void pop() {
        int top = st.top();
        st.pop();
        if(min_st.top() == top){
            min_st.pop();
        }
    }
    int top() {
        return st.top();
    }
    int getMin() {
        return min_st.top();
    }
};

```

## 160 相交链表

[https://leetcode.cn/problems/intersection-of-two-linked-lists/](https://leetcode.cn/problems/intersection-of-two-linked-lists/)

判断两个链表是否相交
便利 两个链表，在达到尾端（注意要达到 nullptr）如果遇到相等的情况，就是相交，否则他们将会在 nullptr 处相交：

```cpp
class Solution {
public:
    ListNode *getIntersectionNode(ListNode *headA, ListNode *headB) {
        if(!headA || !headB) return nullptr;
        auto tpa = headA;
        auto tpb = headB;
        while(tpa!=tpb){
          tpa = tpa? tpa->next : headB;
          tpb = tpb? tpb->next : headA;
        }
        return tpa;
    }
};

```

## 169 多数元素

[https://leetcode.cn/problems/majority-element/](https://leetcode.cn/problems/majority-element/)

1、排序取中间值
2、摩尔投票算法

```cpp
class Solution {
public:
    int majorityElement(vector<int>& nums) {
        int cnt = 0;
        int candidate;
        for(int num: nums){
            if(cnt==0){
                candidate = num;
                cnt++;
            }else{
                if(candidate==num){
                    cnt++;
                }else{
                    cnt--;
                }
            }
        }
        return candidate;
    }
};

```

## 198 打家劫舍

### 打家劫舍 i

[https://leetcode.cn/problems/house-robber/](https://leetcode.cn/problems/house-robber/)

动态规划
max(f[i-1], nums[i]+f[i-2])
偷`i-2`家和`i`家或者偷`i-1`并且不偷`i`家

```cpp
class Solution {
public:
    int rob(vector<int>& nums) {
        vector<int> f(nums.size(), 0);
        if(f.size()==1) return nums[0];
        f[0] = nums[0];
        f[1] = max(nums[0], nums[1]);
        for(int i=2;i<nums.size();i++){
            f[i] = max(f[i-1], nums[i]+f[i-2]);
        }
        return f[nums.size()-1];
    }
};
```

优化空间之后

```cpp
class  Solution {
public:
	int  rob(vector<int>&  nums) {
		int prev = 0;
		int cur = nums[0];
		for(int i=1;i<nums.size();i++)
		{
			int tp = cur;
			cur = max(cur, prev+nums[i]);
			prev = tp;
		}
		return cur;
		}
};
```

### 打家劫舍 ii

[https://leetcode.cn/problems/house-robber-ii/](https://leetcode.cn/problems/house-robber-ii/)

有环 ,如果偷了第 0 家，那么不可以偷第 n-1 家，所以 rob(nums, 0, n-2)，如果偷了 n-1 家，那么不可以偷第 0 家，所以 rob(nums, 1, n-1)。

```cpp
class Solution {
public:
    int rob_helper(vector<int>& nums, int start, int end) {
        int prev = 0;
        int cur = nums[start];
        for (int i = start + 1; i <= end; i++) {
            int tp = cur;
            cur = max(cur, prev + nums[i]);
            prev = tp;
        }
        return cur;
    }
    int rob(vector<int>& nums) {
        if (nums.size() == 1) return nums[0];
        int n = nums.size();
        return max(rob_helper(nums, 1, n - 1), rob_helper(nums, 0, n - 2));
    }
};

```

### 打家劫舍 iii

https://leetcode.cn/problems/house-robber-iii/

核心思想是后序遍历，现将子节点值遍历出来
建立两个哈希表，g[root]表示选择当前节点的最大值，f[root]表示不选择当前节点的最大值

同样的思想，偷到当前节点有两种情况，如果偷当前节点，那么左孩子节点和右孩子节点都不能偷，直接就是`root→val+f[root→left]+f[root→right]`，如果不偷当前节点，那么左孩子可偷可不偷，右孩子也是可偷可不偷，所以`max(g[root→left], f[root→right])+max(g[root→right], f[root→right])`

```cpp
class Solution {
private:
    unordered_map<TreeNode*, int> f; // choose
    unordered_map<TreeNode*, int> g; // not choose
public:
    void dfs(TreeNode* root){
        if(!root){
            return;
        }
        dfs(root->left);
        dfs(root->right);
        f[root] = root->val + g[root->left] + g[root->right];
        g[root] = max(g[root->left], f[root->left]) +  max(g[root->right], f[root->right]);
    }
    int rob(TreeNode* root) {
        dfs(root);
        return max(g[root], f[root]);
    }
};
```

## 200 岛屿数量

https://leetcode.cn/problems/number-of-islands

深度优先遍历+visit 数组

```cpp
class Solution {
private:
    vector<vector<int>> directions = {{1, 0}, {0, 1}, {-1, 0}, {0, -1}};
    vector<vector<bool>> visit;
    int n;
    int m;
public:
    bool is_valid(int i, int j, vector<vector<char>>& grid){
        return i<n && i>=0 && j<m && j>=0 && !visit[i][j] && grid[i][j] == '1';
    }
    void dfs(int i, int j, vector<vector<char>>& grid){
        visit[i][j] = true;
        for(auto& direction : directions){
            int nexti = i + direction[0];
            int nextj = j + direction[1];
            if(is_valid(nexti, nextj, grid)){
                dfs(nexti, nextj, grid);
            }
        }
    }
    int numIslands(vector<vector<char>>& grid) {
        int ans = 0;
        n = grid.size();
        m = grid[0].size();
        visit = vector<vector<bool>> (n, vector<bool>(m));
        for(int i=0;i<n;i++){
            for(int j=0;j<m;j++){
                if(!visit[i][j] && grid[i][j] == '1'){
                    ans++;
                    dfs(i, j, grid);
                }
            }
        }
        return ans;
    }
};

```

## 206 翻转链表

https://leetcode.cn/problems/reverse-linked-list

头插法：正向遍历+dummy 节点

```cpp
class Solution {
public:
    ListNode* reverseList(ListNode* head) {
        auto dummy = new ListNode();
        while(head){
            auto dummy_next = dummy->next;
            auto head_next = head->next;
            dummy->next = head;
            head->next = dummy_next;
            head = head_next;
        }
        return dummy->next;
    }
};
```

迭代法

```cpp
class Solution {
public:
    ListNode* reverseList(ListNode* head) {
        ListNode* prev = nullptr;
        ListNode* cur = head;
        while(cur){
            auto next = cur->next;
            cur->next = prev;
            prev = cur;
            cur = next;
        }
        return prev;

    }
};

```

递归法

```cpp
class Solution {
public:
    ListNode* reverseList(ListNode* head) {
        if(!head || !head->next) return nullptr;
        auto next = head->next;
        auto reversed = reverseList(head->next);
        head->next = nullptr; // 这是重点
        next->next = head;
        return reversed;
    }
};
```

## 207 课程表

https://leetcode.cn/problems/course-schedule

计算入度+广度优先算法

1. 先遍历，记录所有节点的入度和整张图
2. 用 stack 或者 queue push 进入度为 0 的课程（表示可以直接上掉课程），while 循环，每次 pop 出来记录可以上掉的课程，计算此课程的完成是否可以让其他课程进入入度为 0 的状态，如有则 push 进 stack 或者 queue，结果是完成课程的数量是否等于全部课程

```cpp
class Solution {
public:
    bool canFinish(int numCourses, vector<vector<int>>& prerequisites) {
        vector<int> in(numCourses, 0);
        vector<vector<int>> graph(numCourses, vector<int>(numCourses, 0));
        for(auto prerequisite: prerequisites){
            in[prerequisite[0]]++;
            graph[prerequisite[0]][prerequisite[1]] = 1;
        }
        int cnt = 0;
        stack<int> st;
        for(int i=0;i<numCourses;i++){
            if(in[i] == 0){
                st.push(i);
            }
        }
        while(!st.empty())
        {
            auto top = st.top();
            st.pop();
            cnt++;
            for(int i=0;i<numCourses;i++){
                if(graph[i][top] != 0){
                    in[i]--;
                    if(in[i] == 0){
                        st.push(i);
                    }
                }
            }
        }
        return cnt == numCourses;
    }
};

```

## 208 实现前缀树

[https://leetcode.cn/problems/implement-trie-prefix-tree/](https://leetcode.cn/problems/implement-trie-prefix-tree/)

数据类型的主要结构是 children，是一个 vector<Trie\*>, 长度为 26（对应 26 个字母）以及一个 is_end 的 bool 变量代表是否到了结尾。
在插入单词过程中，从 this 指针开始，遍历字符串的每个字符，每个字符串减去'a'，获得下标，如果当前指针的 child 是 nullptr，那么创建新的。遍历到最后一个字母，需要将其 is_end 置 true。
search()和 startwith()通过深度优先搜索，如果是 nullptr 那么就没有包含这个单词，search 比 startwith 条件更加苛刻，需要 is_end 为 true

```cpp
class Trie {
private:
    vector<Trie*> cache;
    bool is_end;
public:
    Trie(): cache(26), is_end(false) {
        is_end = false;
    }
    Trie* dfs(const string& word){
        auto cur = this;
        for(char c: word){
            int index = c - 'a';
            if(!cur->cache[index]){
                return nullptr;
            }
            cur = cur->cache[index];
        }
        return cur;
    }
    void insert(string word) {
        auto cur = this;
        for(char c:word){
            int index = c - 'a';
            if(!cur->cache[index]){
                cur->cache[index] = new Trie();
            }
            cur = cur->cache[index];
        }
        cur->is_end = true;
    }

    bool search(string word) {
        auto cur = dfs(word);
        return cur!=nullptr && cur->is_end;
    }

    bool startsWith(string prefix) {
        auto cur = dfs(prefix);
        return cur!=nullptr;
    }
};
```

## 215 数组中的第 k 大元素

[https://leetcode.cn/problems/kth-largest-element-in-an-array/](https://leetcode.cn/problems/kth-largest-element-in-an-array/)

1、快速排序的变形
在快速排序中，如果 pivot 就是第 k 个，就得到了最终的结果，如果 pivot 比 k 要小，那么快排 pivot+1 到 end，如果 pivot 比 k 要大，那么快排 begin 到 pivot-1

```cpp
class Solution {
private:
    int k;
public:
    int partition(vector<int>& nums, int left, int right){
        int i = left-1;
        int pivot_val = nums[right];
        for(int j=left;j<right;j++){
            if(nums[j] < pivot_val){
                i++;
                swap(nums[j], nums[i]);
            }
        }
        i++;
        swap(nums[i], nums[right]);
        return i;
    }
    int quick_sort(vector<int>& nums, int left, int right){
        int pivot = partition(nums, left, right);
        if(pivot == k){
            return nums[pivot];
        }else if(pivot < k){
            return quick_sort(nums, pivot+1, right);
        }
        return quick_sort(nums, left, pivot-1);
    }
    int findKthLargest(vector<int>& nums, int k) {
        this->k = nums.size() - k ;
        return quick_sort(nums, 0, nums.size()-1);
    }
};

```

2、堆排序
用大根堆，大根堆第一个元素是最大值，进行 k 次删除堆顶元素之后就是答案

```cpp
class Solution {
public:
    void heapify(vector<int>& nums, int i, int n){
        int left = i*2+1;
        int right = i*2+2;
        int largest = i;
        if(left < n && nums[left]> nums[largest] ){
            largest = left;
        }
        if(right < n && nums[right] > nums[largest]){
            largest = right;
        }
        if(largest!=i){
            swap(nums[largest], nums[i]);
            heapify(nums, largest, n);
        }
    }
    void build_heap(vector<int>& nums){
        for(int i=(nums.size()-2)/2;i>=0;i--){
            heapify(nums, i, nums.size());
        }
    }
    int findKthLargest(vector<int>& nums, int k) {
        build_heap(nums);
        int n = nums.size();
        for(int i=0;i<k-1;i++){
            swap(nums[0], nums[n-1]);
            n--;
            heapify(nums, 0, n);
        }
        return nums[0];
    }
};

```

## 221.最大正方形

[https://leetcode.cn/problems/maximal-square/](https://leetcode.cn/problems/maximal-square/)

`f[i][j] = min(f[i-1][j-1], min(f[i-1][j], f[i][j-1])) + 1;`

```cpp
class Solution {
public:
    int maximalSquare(vector<vector<char>>& matrix) {
        int n = matrix.size();
        int m = matrix[0].size();
        int res = 0;
        vector<vector<int>> f(n, vector<int>(m, 0));
        for(int i=0;i<n;i++)
        {
            f[i][0] = matrix[i][0] == '1'? 1: 0;
            res = max(res, f[i][0]);
        }
        for(int i=0;i<m;i++)
        {
            f[0][i] = matrix[0][i] == '1'? 1: 0;
            res = max(res, f[0][i]);
        }
        for(int i=1;i<n;i++)
        {
            for(int j=1;j<m;j++)
            {
                if(matrix[i][j] == '1')
                {
                    f[i][j] = min(f[i-1][j-1], min(f[i-1][j], f[i][j-1])) + 1;
                    res = max(res, f[i][j]);
                }
            }
        }
        return res*res;
    }
};

```

## 226 翻转二叉树

[https://leetcode.cn/problems/invert-binary-tree/](https://leetcode.cn/problems/invert-binary-tree/)

1、递归方法

```cpp
class Solution {
public:
    TreeNode* invertTree(TreeNode* root) {
        if(!root) return root;
        auto left = root->left;
        auto right = root->right;
        invertTree(root->left);
        invertTree(root->right);
        root->left = right;
        root->right = left;
        return root;
    }
};
```

2、非递归
用 stack 或者 queue 进行储存（两种容器的代码是一样的，只不过或是层序遍历，stack 是深度优先遍历）

```cpp
class Solution {
public:
    TreeNode* invertTree(TreeNode* root) {
        if(root==nullptr){
            return nullptr;
        }
        queue<TreeNode*> q;
        q.push(root);
        while(!q.empty())
        {
            auto front = q.top();
            q.pop();
            auto left = front->left;
            front->left = front->right;
            front->right = left;
            if(front->left){
                q.push(front->left);
            }
            if(front->right){
                q.push(front->right);
            }
        }
        return root;

    }
};

```

## 234 回文链表

[https://leetcode.cn/problems/palindrome-linked-list/](https://leetcode.cn/problems/palindrome-linked-list/)

1、用 vector 储存

```cpp
class Solution {
public:
    bool isPalindrome(ListNode* head) {
        vector<ListNode*> v;
        while(head){
            v.push_back(head);
            head = head->next;
        }
        for(int i=0, j=v.size()-1;i<j;i++, j--){
            if(v[i]->val != v[j]->val){
                return false;
            }
        }
        return true;
    }
};

```

2、找到中点，翻转链表。 奇数链表中间点被看做前半部分，判断比较前半部分和翻转的后半部分是否相同时，以后半部分为标准。

```cpp
class Solution {
public:
    ListNode* reverse_list(ListNode* head){
        ListNode* prev = nullptr;
        while(head){
            auto next = head->next;
            head->next = prev;
            prev = head;
            head = next;
        }
        return prev;
    }
    bool isPalindrome(ListNode* head) {
        auto fast = head;
        auto slow = head;
        while(fast->next && fast->next->next){
            fast = fast->next->next;
            slow = slow->next;
        }
        auto mid = slow->next;
        slow->next = nullptr;
        mid = reverse_list(mid);
        while(mid && head){
            if(mid->val != head->val){
                return false;
            }
            mid = mid->next;
            head = head->next;
        }
        return true;
    }
};
```

## 236 二叉树的公共祖先

[https://leetcode.cn/problems/lowest-common-ancestor-of-a-binary-tree/](https://leetcode.cn/problems/lowest-common-ancestor-of-a-binary-tree/)

1、遍历方法
深度搜索函数搜查是否包含 q 和 p，在搜索过程中，如果 root 等于 q 或者 p 并且 left 和 right 某个为正，或者左右都为正，代表是公共祖先

```cpp
class Solution {
private:
    TreeNode* res;
public:
    bool contains(TreeNode* root, TreeNode* p, TreeNode* q){
        if(root == nullptr) return false;
        bool left = contains(root->left, p, q);
        bool right = contains(root->right, p, q);
        if( ( root==p || root==q) && (left || right) || (left && right)){
            res = root;
            return true;
        }
        return root==p || root == q ||left || right;
    }
    TreeNode* lowestCommonAncestor(TreeNode* root, TreeNode* p, TreeNode* q) {
        contains(root, p, q);
        return res;
    }
};
```

2、递归
递归函数返回值为 nullptr 表示没有找到

```cpp
class Solution {
public:
    TreeNode* lowestCommonAncestor(TreeNode* root, TreeNode* p, TreeNode* q) {
        if(root == nullptr || root==p || root==q) return root;
        auto left = lowestCommonAncestor(root->left, p, q);
        auto right = lowestCommonAncestor(root->right, p, q);
        if(left && right) return root;
        if(left != nullptr)
        return left;
        return right;
    }
};
```

## 238 除自身以为的数组的乘积

[https://leetcode.cn/problems/product-of-array-except-self/](https://leetcode.cn/problems/product-of-array-except-self/)

维护两个数组，一个是左边累积；一个是右边累积

```
class Solution {
public:
    vector<int> productExceptSelf(vector<int>& nums) {
        vector<int> res(nums.size(), 1);
        vector<int> left(nums.size(), 1);
        vector<int> right(nums.size(), 1);
        for(int i=1;i<nums.size();i++)
        {
            left[i] = nums[i-1]*left[i-1];
        }
        for(int i=nums.size()-2;i>=0;i--)
        {
            right[i] = nums[i+1]*right[i+1];
        }
        for(int i=1;i<nums.size()-1;i++)
        {
            res[i] = left[i]*right[i];
        }
        res[0] = right[0];
        res[nums.size()-1] = left[nums.size()-1];
        return res;
    }
};

```

可以用一个变量代替第二个数组，因为不用保留前面的变量。

## 239 滑动窗口的最大值

[https://leetcode.cn/problems/sliding-window-maximum/](https://leetcode.cn/problems/sliding-window-maximum/)

1、优先队列
维护一个优先队列，内容是元素的值和下标的 pair，每次 push 元素之前判断队列顶元素下标是否在范围内。

```cpp
class Solution {
public:
    struct node{
        int idx;
        int val;
    };
    struct cmp{
        bool operator ()(node& a, node& b){
            return a.val < b.val;
        }
    };
    vector<int> maxSlidingWindow(vector<int>& nums, int k) {
        priority_queue<node, vector<node>, cmp> q;
        vector<int> res;
        for(int i=0;i<k;i++)
        {
            q.push(node{i, nums[i]});
        }
        res.push_back(q.top().val);
        for(int i=k;i<nums.size();i++)
        {
            q.push(node{i, nums[i]});
            while(q.top().idx <= i-k){
                q.pop();
            }
            res.push_back(q.top().val);
        }
        return res;
    }
};
```

2、优化——单调队列
维护一个双端队列， 里面存元素的下标，让队列前面元素比后面元素大，且开头元素一定要符合窗口的范围。

```cpp
class Solution {
public:
    vector<int> maxSlidingWindow(vector<int>& nums, int k) {
        deque<int> dq;
        vector<int> res;
        for (int i = 0; i < k; i++) {
            while (!dq.empty() && nums[i] > nums[dq.back()]) {
                dq.pop_back();
            }
            dq.push_back(i);
        }
        res.push_back(nums[dq.front()]);
        for (int i = k; i < nums.size(); i++) {
            while (!dq.empty() && dq.front() <= i - k) {
                dq.pop_front();
            }
            while (!dq.empty() && nums[dq.back()] <= nums[i]) {
                dq.pop_back();
            }
            dq.push_back(i);
            res.push_back(nums[dq.front()]);
        }
        return res;
    }
};

```

## 240 搜索二维矩阵

[https://leetcode.cn/problems/search-a-2d-matrix-ii/](https://leetcode.cn/problems/search-a-2d-matrix-ii/)

从右上开始搜索

```cpp
class Solution
{
public:
    bool searchMatrix(vector<vector<int>>& matrix, int target) {
        int n = matrix.size();
        int m = matrix[0].size();
        int i = 0;
        int j = m - 1;
        while (i < n && j >= 0) {
            int val = matrix[i][j];
            if (val == target) {
                return true;
            }
            else if (val > target) {
                j--;
            }
            else {
                i++;
            }

        }
        return false;
    }
};

```

## 279 完全平方数

动态规划 f[i] = min(f[i], f[i-j*j]+1)，其中(j\*j≤i)

```cpp
class Solution {
public:
    int numSquares(int n) {
        vector<int> f(n+1, INT_MAX);
        f[0] = 0;
        for(int i=1;i<=n;i++)
        {
            for(int j=1;j<=sqrt(i);j++)
            {
                f[i] = min(f[i], f[i-j*j]+1);
            }
        }
        return f[n];
    }
};
```

## 283 移动零

[https://leetcode.cn/problems/move-zeroes/](https://leetcode.cn/problems/move-zeroes/)

和快速排序的 parition 差不多

```cpp
class Solution {
public:
    void moveZeroes(vector<int>& nums) {
        int idx = -1;
        for(int i=0;i<nums.size();i++){
            if(nums[i] !=0){
                ++idx;
                swap(nums[idx], nums[i]);
            }
        }
    }
};

```

## 287 寻找重复数字

[https://leetcode.cn/problems/find-the-duplicate-number/](https://leetcode.cn/problems/find-the-duplicate-number/)

1、二分法 O（n\*lgn） 每次 n 次循环计算

```cpp
class Solution {
public:
    int findDuplicate(vector<int>& nums) {
        int l = 1;
        int r = nums.size() - 1;
        int cnt;
        while (l < r) {
            int mid = l + (r - l) / 2;
            cnt = 0;
            for (int i = 0; i < nums.size(); i++) {
                if (nums[i] <= mid) {
                    cnt++;
                }
            }
            if (cnt > mid) {
                r = mid;
            }
            else {
                l = mid + 1;
            }
        }
        return l;
    }
};
```

2、快慢指针
因为一定有某个点有两条线指向自己，所以必定存在环，且环的入口就是重复的数字。同链表的环

```cpp
class Solution {
public:
    int findDuplicate(vector<int>& nums) {
        int fast = 0;
        int slow = 0;
        fast = nums[nums[fast]];
        slow = nums[slow];
        while (nums[fast] != nums[slow]) {
            fast = nums[nums[fast]];
            slow = nums[slow];
        }
        int cur = 0;
        while (nums[cur] != nums[slow]) {
            cur = nums[cur];
            slow = nums[slow];
        }
        return nums[slow];
    }
};

```

## 297 二叉树的序列化和反序列化

[https://leetcode.cn/problems/serialize-and-deserialize-binary-tree/](https://leetcode.cn/problems/serialize-and-deserialize-binary-tree/)

注意字符串拼接的时候要用+= ，c++底层有优化。
1、层序遍历

```cpp
class Codec {
public:
   vector<string> get_nodes(string data) {
       vector<string> res;
       int i = 0;
       while (i < data.size()) {
           int j = i;
           while (j < data.size() && data[j] != ',') {
               j++;
           }
           res.push_back(data.substr(i, j - i));
           i = j + 1;
       }
       return res;
   }

   // Encodes a tree to a single string.
   string serialize(TreeNode* root) {
       string data = "";
       if (root == nullptr) return data;
       queue<TreeNode*> q;
       q.push(root);
       while (!q.empty()) {
           auto front = q.front();
           q.pop();
           if (front) {
               data += to_string(front->val) + ",";
               q.push(front->left);
               q.push(front->right);
           }
           else {
               data += "null,";
           }
       }
       return data.substr(0, data.size() - 1);
   }

   // Decodes your encoded data to tree.
   TreeNode* deserialize(string data) {
       if (data.size() == 0) return nullptr;
       auto nodes = get_nodes(data);
       queue<TreeNode*> q;
       auto root = new TreeNode(stoi(nodes[0]));
       q.push(root);
       int idx = 1;
       while (!q.empty()) {
           auto front = q.front();
           q.pop();
           if (nodes[idx] != "null") {
               front->left = new TreeNode(stoi(nodes[idx]));
               q.push(front->left);
           }
           idx++;
           if (nodes[idx] != "null") {
               front->right = new TreeNode(stoi(nodes[idx]));
               q.push(front->right);
           }
           idx++;
       }
       return root;
   }
};
```

## 300 最长递增子序列

[https://leetcode.cn/problems/longest-increasing-subsequence/](https://leetcode.cn/problems/longest-increasing-subsequence/)

动态规划，要注意把之前的也加上

```cpp
class Solution {
public:
   int lengthOfLIS(vector<int>& nums) {
       vector<int> f(nums.size(), 1);
       int res = 1;
       for (int i = 1; i < nums.size(); i++) {
           for (int j = 0; j < i; j++) {
               if (nums[i] > nums[j]) {
                   f[i] = max(f[i], f[j] + 1);
               }
           }
           res = max(f[i], res);
       }
       return res;
   }
};
```

## 301 删除无效括号

[https://leetcode.cn/problems/remove-invalid-parentheses/](https://leetcode.cn/problems/remove-invalid-parentheses/)

先计算需要删除的左括号和右括号，然后通过深度优先搜索遍历

```cpp
class Solution {
private:
    vector<string> res;
public:
    bool is_pair(char a, char b){
        if(a==')' && b=='(') return true;
        return false;
    }
    bool is_valid(string s){
        stack<char> stk;
        for(char c: s){
            if(c!=')' && c!='(') continue;
            if(!stk.empty() && is_pair(c, stk.top())){
                stk.pop();
            }else{
                stk.push(c);
            }
        }
        return stk.empty();
    }

    void dfs(string s, int lr, int rr, int idx){
        if(lr==0 && rr == 0 && is_valid(s)){
            res.push_back(s);
            return;
        }
        for(int i=idx;i<s.size();i++){
            if(i!=0 && s[i] == s[i-1]) continue;
            if(lr+rr > s.size()-i) return;
            if(s[i] == '(' && lr>0){
                dfs(s.substr(0, i)+s.substr(i+1, s.size()-i-1), lr-1, rr, i);

            }
            if(s[i] == ')' && rr>0){
                dfs(s.substr(0, i)+s.substr(i+1, s.size()-i-1), lr, rr-1, i);
            }
        }
    }
    vector<string> removeInvalidParentheses(string s) {
        int lr = 0;
        int rr = 0;
        for(char c: s){
            if(c=='('){
                lr++;
            }else if(c==')'){
                if(lr>0){
                    lr--;
                }else{
                    rr++;
                }
            }
        }
        dfs(s, lr, rr, 0);
        return res;
    }
};
```

### 20 有效括号

[https://leetcode.cn/problems/valid-parentheses/submissions/](https://leetcode.cn/problems/valid-parentheses/submissions/)
用 stack 进行判断，最后判断 stack 是否为空

```
class Solution {
public:
   bool is_pair(char a, char b) {
       if (a == ')' && b == '(') return true;
       if (a == ']' && b == '[') return true;
       if (a == '}' && b == '{') return true;
       return false;
   }
   bool isValid(string s) {
       stack<char> st;
       for (auto c : s) {
           if (!st.empty() && is_pair(c, st.top()) == true) {
               st.pop();
           }
           else {
               st.push(c);
           }
       }
       return st.empty();
   }
};

```

### 22 括号生成

[https://leetcode.cn/problems/generate-parentheses/](https://leetcode.cn/problems/generate-parentheses/)

回溯+剪枝
左括号数量比右括号数量多

```cpp
class Solution {
private:
   vector<string> res;
public:
   void dfs(string& str, int left, int right, int n) {
       if (left > n || right > n) return;
       if (left == n && right == n) {
           res.push_back(str);
       }
       if (left > right) {
           str.push_back('(');
           dfs(str, left + 1, right, n);
           str.pop_back();
           str.push_back(')');
           dfs(str, left, right + 1, n);
           str.pop_back();
       }
       else if (left == right) {
           str.push_back('(');
           dfs(str, left + 1, right, n);
           str.pop_back();
       }
   }
   vector<string> generateParenthesis(int n) {
       int left = 0;
       int right = 0;
       string str = "";
       dfs(str, left, right, n);
       return res;
   }
};
```

## 309 买卖股票

### 121 买卖股票的最佳时机

[https://leetcode.cn/problems/best-time-to-buy-and-sell-stock/](https://leetcode.cn/problems/best-time-to-buy-and-sell-stock/)

维护一个变量，记录到当前的最小值

```cpp
class  Solution {
public:
		int  maxProfit(vector<int>&  prices) {
				int curMin = INT_MAX;
				int res = 0;
				for(int price:prices){
						res = max(res, price-curMin);
						curMin = min(curMin, price);
					}
					return res;
		}
};
```

### 122. 买卖股票的最佳时机 II

[https://leetcode.cn/problems/best-time-to-buy-and-sell-stock-ii/](https://leetcode.cn/problems/best-time-to-buy-and-sell-stock-ii/)

不能多次交易股票，在 i 天结束后，只存在两个状态，0 是持有现金，1 是持有股票

```cpp
f[0][i] = max(f[0][i-1], f[1][i-1]+price[i])
f[1][i] = max(f[1][i-1], f[0][i-1]-price[i])
```

i 天如果持有现金，要么 i 天没有买股票，那么取 i-1 天持有现金，或者 i 天卖出股票
i 天如果持有股票， 那么 i 天没有买股票，那么取 i-1 天持有股票，或者 i 天买入股票

```cpp
class Solution
{
	public:
		int maxProfit(vector<int> &prices)
		{
			// 动态规划
			vector<vector < int>> f(2, vector<int> (prices.size(), 0));
			int cash_hold = 0;
			int stock_hold = -prices[0];
			for (int i = 1; i < prices.size(); i++)
			{
				cash_hold = max(cash_hold, stock_hold + prices[i]);
				stock_hold = max(stock_hold, cash_hold - prices[i]);
			}

			return cash_hold;
		}
};

```

### 309 最佳买卖股票时机含冷冻期

[https://leetcode.cn/problems/best-time-to-buy-and-sell-stock-with-cooldown/](https://leetcode.cn/problems/best-time-to-buy-and-sell-stock-with-cooldown/)

有冷却期，那么 i 天结束后就有三种状态：1、持有现金，非冷却期。2、持有现金，冷却期 3、持有股票

```cpp
class Solution {
public:
   int maxProfit(vector<int>& prices) {
       if (prices.size() == 1 || prices.size() == 0)return 0;
       vector<vector<int>> f(3, vector<int>(prices.size()));
       // f[0][i] 持有现金，非冷冻期
       // f[1][i] 持有现金，冷冻期
       // f[2][i] 持有股票
       f[2][0] = -prices[0];
       for (int i = 1; i < prices.size(); i++)
       {
           f[0][i] = max(f[0][i - 1], f[1][i - 1]);
           f[1][i] = f[2][i - 1] + prices[i];
           f[2][i] = max(f[2][i - 1], f[0][i - 1] - prices[i]);
       }
       return max(f[0][prices.size() - 1], f[1][prices.size() - 1]);
   }
};
```

## 312.戳气球
https://leetcode.cn/problems/burst-balloons/
动态规划要求子问题必须独立，所以我们将其 nums 扩容 2（两个边界-1 和 n）并设置成 1，气球的索引变成了 1-n,并且我们将戳气球的操作变成添加气球操作

[题解](https://mp.weixin.qq.com/s?__biz=MzAxODQxMDM0Mw==&mid=2247485172&idx=1&sn=b860476b205b04f960ea0de6f70d3553&scene=21#wechat_redirect)

动态规划

```
dp[i][j]
```

表示戳破在 i 和 j 之间的气球可以获得的最大值，其中不包括 i 和 j，这样以来就转换成求

```
dp[0][n+1]
```

如果正向思考，那么就是回到回溯。我们需要反向思考，

**谁是最后一个被戳破的气球**

，那就问题就可以被转换成

```
dp[i][j] = max(d[i][k]+d[k][j] + points[i]*point[j]*point[k])
```

```
d[i][j]
```

依赖的状态是

```
d[i][k]
```

和

```
d[k][j]
```

(其中 i<k<j)，所以要注意顺序，


```cpp
class Solution {
public:
    int maxCoins(vector<int>& nums) {
        int n = nums.size();
        vector<int> val(n + 2, 1);
        for (int i = 0; i < nums.size(); i++)
        {
            val[i + 1] = nums[i];
        }
        vector<vector<int>> dp(n + 2, vector<int>(n + 2, 0));
        for (int i = n - 1; i >= 0; i--)
        {
            for (int j = i + 1; j <= n + 1; j++)
            {
                for (int k = i + 1; k < j; k++)
                {
                    int sum = dp[i][k] + dp[k][j] + val[i] * val[j] * val[k];
                    dp[i][j] = max(dp[i][j], sum);
                }
            }
        }
        return dp[0][n + 1];
    }
};
```

## 322 零钱兑换

[https://leetcode.cn/problems/coin-change/](https://leetcode.cn/problems/coin-change/)
动态规划
f[i] = min(f[i-coin]+1, f[i])

```
class Solution {
public:
    int coinChange(vector<int>& coins, int amount) {
        int max_val = amount + 1;
        vector<int> f(amount + 1, max_val);
        f[0] = 0;
        for (int i = 0; i <= amount; i++) {
            for (int coin : coins) {
                if (i - coin >= 0) {
                    f[i] = min(f[i], f[i - coin] + 1);
                }
            }
        }
        return f[amount] != max_val ? f[amount] : -1;
    }
};
```

## 338 比特位计算

[https://leetcode.cn/problems/counting-bits/](https://leetcode.cn/problems/counting-bits/)

动态规划， 如果 i 是偶数，那么比特位就是 i/2，如果是奇数，那么就是 f[i-1]+1
因为偶数的话相当于末尾补了一个 0, 如 2：`10` 4：`100`
奇数的话就相当于上一个偶数添加了一个 1, 如 2:`10` 3:`11`

```cpp
class Solution {
public:
    vector<int> countBits(int n) {
        vector<int> f(n + 1);
        f[0] = 0;
        for (int i = 1; i <= n; i++) {
            if (i % 2 == 0) {
                f[i] = f[i / 2];
            }
            else {
                f[i] = f[i - 1] + 1;
            }
        }
        return f;
    }
};
```

## 347 前 k 个高频元素

[https://leetcode.cn/problems/kth-largest-element-in-an-array/solution/](https://leetcode.cn/problems/top-k-frequent-elements/)

先用 unordered_map 进行存储，之后对频率进行操作，类似于第 k 个大的元素
1、优先队列

```cpp
class Solution {
private:
    vector<int> res;
public:
    struct node {
        int val;
        int freq;
    };
    struct cmp {
        bool operator()(node a, node b) {
            return a.freq < b.freq;
        }
    };
    vector<int> topKFrequent(vector<int>& nums, int k) {
        priority_queue<node, vector<node>, cmp> pq;
        unordered_map<int, int> dict;
        for (int each : nums) {
            dict[each]++;
        }
        for (auto it = dict.begin(); it != dict.end(); it++) {
            pq.push(node{ it->first, it->second });
        }
        for (int i = 0; i < k; i++) {
            auto front = pq.top();
            pq.pop();
            res.push_back(front.val);
        }
        return res;
    }
};
```

## ! 394. 字符串解码

使用栈

[https://leetcode.cn/problems/decode-string/?favorite=2cktkvj](https://leetcode.cn/problems/decode-string/)

```cpp
class Solution {
public:
    string decodeString(string s) {
        string ans = "";
        int mul = 0;
        stack<pair<int, string>> stk;
        for(int i=0;i<s.size();i++){
            char cur = s[i];
            if(isdigit(cur)){
                mul = mul*10 + cur-'0';
            }else if(isalpha(cur)){
                if(stk.empty()){
                    ans.push_back(cur);
                }else{
                    stk.top().second.push_back(cur);
                }
            }else if(cur == '['){
                stk.push({mul, ""});
                mul = 0;
            }else if(cur == ']'){
                auto top = stk.top();
                stk.pop();
                string tp = "";
                for(int i=0;i<top.first;i++){
                    tp += top.second;
                }
                if(stk.empty()){
                    ans += tp;
                }else{
                    stk.top().second += (tp);
                }
            }
        }
        return ans;
    }
};
```

## 406 根据身高建立队列

[https://leetcode.cn/problems/queue-reconstruction-by-height/](https://leetcode.cn/problems/queue-reconstruction-by-height/)
cmp 函数，第一个顺序是身高（高的排前面），第二顺序前面的人数（人少的排前面）
然后 for 循环遍历插入，insert 位置是 vector.begin()+前面排的人数

```cpp
class Solution {
public:
    static bool cmp(vector<int> a, vector<int> b){
        if(a[0] != b[0]) return a[0] > b[0];
        return a[1] < b[1];
    }
    vector<vector<int>> reconstructQueue(vector<vector<int>>& people) {
        vector<vector<int>> res;
        sort(people.begin(), people.end(), cmp);
        for(int i=0;i<people.size();i++){
            res.insert(res.begin() + people[i][1], people[i]);
        }
        return res;
    }
};

```

## 416 分割等和子集

[https://leetcode.cn/problems/partition-equal-subset-sum/](https://leetcode.cn/problems/partition-equal-subset-sum/)

题目要求是是否存在等分子集，首先算出 sum，然后转化为背包问题，也就是可以可以完全放下 sum/2 的物品

```cpp
class Solution {
public:
    bool canPartition(vector<int>& nums) {
        int sum = 0;
        for(int val: nums){
            sum += val;
        }
        if(sum%2 != 0) return false;
        int target = sum/2;
        int n = nums.size();
        vector<vector<int>> dp(n+1, vector<int>(target+1, false));
        for(int i=0;i<=n;i++)
        {
            dp[i][0] = true;
        }
        for(int i=1;i<=n;i++)
        {
            for(int j=1;j<=target;j++)
            {
                if(j-nums[i-1] < 0)
                {
                    dp[i][j] = dp[i-1][j];
                }else{
                    if(j-nums[i-1] == 0){
                        dp[i][j] = true;
                    }else{
                        dp[i][j] = dp[i-1][j-nums[i-1]] || dp[i-1][j];
                    }
                }
            }
        }
        return dp[n][target];
    }
};
```

## 437 路径总和

[https://leetcode.cn/problems/path-sum-iii/](https://leetcode.cn/problems/path-sum-iii/)

前缀和, dfs 回溯

```cpp
class Solution {
private:
    unordered_map<long long, int> cache;
    int res = 0;
public:
    void dfs(TreeNode* root, long long cur, int targetSum){
        cur += root->val;
        res += cache[cur-targetSum];
        cache[cur]++;
        if(root->left)
            dfs(root->left, cur, targetSum);
        if(root->right)
            dfs(root->right, cur, targetSum);
        cache[cur]--;
    }
    int pathSum(TreeNode* root, int targetSum) {
        if(root == nullptr) return 0;
        cache[0] = 1;
        dfs(root, 0, targetSum);
        return res;
    }
};
```

## 438 找到字符串中所有字母异位词
https://leetcode.cn/problems/find-all-anagrams-in-a-string/

滑动窗口,注意滑动窗口 valid 的条件，当 windows[add]==need[add]是 valid 才增加或者减少，valid 的数值和 need 的 key 值相同时进行左边界改变

```cpp
class Solution {
public:
    vector<int> findAnagrams(string s, string p) {
        unordered_map<char, int> need;
        unordered_map<char, int> window;
        for(char c: p){
            need[c]++;
        }
        vector<int> res;
        int left = 0;
        int right = 0;
        int valid = 0;
        while(right<s.size())
        {
            char add = s[right++];
            if(need.count(add))
            {
                window[add]++;
                if(window[add] == need[add]){
                    valid++;
                }
            }
            while(valid >= need.size())
            {
                char deleted = s[left];
                if(right-left==p.size()){
                    res.push_back(left);
                }
                if(need.count(deleted))
                {
                    if(window[deleted] == need[deleted]){
                        valid--;
                    }
                    window[deleted]--;
                }
                left++;
            }
        }
        return res;
    }
};
```

## 560 和为 K 的子数组

[https://leetcode.cn/problems/subarray-sum-equals-k/](https://leetcode.cn/problems/subarray-sum-equals-k/)

```cpp
class Solution {
public:
    int subarraySum(vector<int>& nums, int k) {
        unordered_map<long long, int> cache;
        cache[0] = 1;
        int cur = 0;
        int res = 0;
        for(int i=0;i<nums.size();i++)
        {
            cur += nums[i];
            res += cache[cur-k];
            cache[cur]++;
        }
        return res;
    }
};
```

# 448 找到所有数组中消失的数字

[https://leetcode.cn/problems/find-all-numbers-disappeared-in-an-array/](https://leetcode.cn/problems/find-all-numbers-disappeared-in-an-array/)

tricky， 遍历数组， num[i]的 index 对应的数字加上数组的长度，最后小于等于数组长度的就是消失的。

```cpp
class Solution {
public:
    vector<int> findDisappearedNumbers(vector<int>& nums) {
        int n = nums.size();
        for(int i=0;i<nums.size();i++)
        {
            nums[(nums[i]-1)%n] += n;
        }
        vector<int> res;
        for(int i=0;i<nums.size();i++){
            if(nums[i] <=n){
                res.push_back(i+1);
            }
        }
        return res;
    }
};
```

## 461 汉明距离
https://leetcode.cn/problems/hamming-distance/

异或

```cpp
class Solution {
public:
    int hammingDistance(int x, int y) {
        int t = x^y;
        int op = 1;
        int res = 0;
        while(t)
        {
            res += t&op;
            t = t>>1;
        }
        return res;
    }
};
```

# 494 目标和

[https://leetcode.cn/problems/target-sum](https://leetcode.cn/problems/target-sum)


算出 neg 的和然后转化为背包问题

```cpp
class Solution {
public:
    int findTargetSumWays(vector<int>& nums, int target) {
        int sum = 0;
        for(int num: nums){
            sum += num;
        }
        // neg + pos = sum
        // -neg + pos = target
        int tp = sum - target;
        if(tp %2 != 0 || tp<0){
            return 0;
        }
        int neg = tp/2;
        int n = nums.size();
        vector<vector<int>> dp(n+1, vector<int>(neg+1, 0));
        dp[0][0] = 1;
        for(int i=1;i<=n;i++)
        {
            for(int j=0;j<=neg;j++)
            {
                int cur = nums[i-1];
                if(j- cur < 0){
                    dp[i][j] = dp[i-1][j];
                }else{
                    dp[i][j] = dp[i-1][j] + dp[i-1][j-cur]; // 不选：计算i-1，j的数量，选了：计算i-1， j-cur的数量；
                }
            }
        }
        return dp[n][neg];
    }
};
```

# 538. 把二叉搜索树转换为累加树

深度搜索，不过是先右节点，中间节点，左节点，用全局的 cur 变量记录

```cpp
class Solution {
private:
    int cur;
public:
    void dfs(TreeNode* root){
        if(root==nullptr) return;
        dfs(root->right);
        cur += root->val;
        root->val = cur;
        dfs(root->left);
    }
    TreeNode* convertBST(TreeNode* root) {
        dfs(root);
        return root;
    }
};
```

## 543 二叉树直径

[https://leetcode.cn/problems/diameter-of-binary-tree](https://leetcode.cn/problems/diameter-of-binary-tree/)

定义 max_depth 函数，在计算最大深度中顺便计算本题结果。

```cpp
class Solution {
private:
    int res = INT_MIN;
public:
    int max_depth(TreeNode* root){
        if(root == nullptr) return 0;
        int left = max_depth(root->left);
        int right = max_depth(root->right);
        res = max(res, left+right);
        return 1+max(left, right);
    }
    int diameterOfBinaryTree(TreeNode* root) {
        if(root == nullptr) return 0;
        max_depth(root);
        return res;
    }
};
```

## 560 和为 K 的子数组

[https://leetcode.cn/problems/subarray-sum-equals-k](https://leetcode.cn/problems/subarray-sum-equals-k/?favorite=2cktkvj)

前缀和

```cpp
class Solution {
public:
    int subarraySum(vector<int>& nums, int k) {
        unordered_map<long long, int> cache;
        cache[0] = 1;
        int cur = 0;
        int res = 0;
        for(int i=0;i<nums.size();i++)
        {
            cur += nums[i];
            res += cache[cur-k];
            cache[cur]++;
        }
        return res;
    }
};
```

## 581 最短无序连续子数组

[https://leetcode.cn/problems/shortest-unsorted-continuous-subarray](https://leetcode.cn/problems/shortest-unsorted-continuous-subarray/)

分为三段左段，中段和右段。左端和右端是标准的升序数组，中段无序，但是满足其最小值大于左段最大值，最大值小于右端最小值。我们分别从左边开始遍历和从右边开始遍历，左边开始遍历寻找中段 end，具体是，如果遍历到的点比最大值还要小那么更新 end（原理是右端总是有当前值就是最大值），同理右边开始遍历，寻找中段 begin，如果遍历到点比最小值还小那么更新 begin（原理是左端总是有当前值就是最小值）

```cpp
class Solution {
public:
    int findUnsortedSubarray(vector<int>& nums) {
        int end = -1;
        int max_val = nums[0];
        for(int i=0;i<nums.size();i++){
            max_val = max(max_val, nums[i]);
            if(nums[i] < max_val){
                end = i;
            }
        }
        int start = -1;
        int min_val = nums[nums.size()-1];
        for(int i = nums.size()-1;i>=0;i--){
            min_val = min(min_val, nums[i]);
            if(nums[i]>min_val){
                start = i;
            }
        }
        return start==-1? 0: end-start+1;
    }
};
```

##

## 617 合并二叉树

[https://leetcode.cn/problems/merge-two-binary-trees](https://leetcode.cn/problems/merge-two-binary-trees)

```cpp
class Solution {
public:
    TreeNode* mergeTrees(TreeNode* root1, TreeNode* root2) {
        if(root1 == nullptr) return root2;
        if(root2 == nullptr) return root1;
        root1->val = root1->val + root2->val;
        root1->left = mergeTrees(root1->left, root2->left);
        root1->right = mergeTrees(root1->right, root2->right);
        return root1;
    }
};
```

## 647 回文子串

[https://leetcode.cn/problems/palindromic-substrings](https://leetcode.cn/problems/palindromic-substrings/)

中心拓展法

```cpp
class Solution {
private:
    int res = 0;
public:
    void extend_center(string& s, int start, int end)
    {
        while(start>=0 && end<s.size() && s[start] == s[end])
        {
            res++;
            start--;
            end++;
        }
    }

    int countSubstrings(string s) {
        for(int i=0;i<s.size();i++)
        {
            extend_center(s, i, i+1);
            extend_center(s, i, i);
        }
        return res;
    }
};
```

## 739 每日温度

https://leetcode.cn/problems/daily-temperatures/

单调栈

```cpp
class Solution {
public:
    vector<int> dailyTemperatures(vector<int>& temperatures) {
        stack<int> st;
        vector<int> res(temperatures.size(), 0);
        for(int i=0;i<temperatures.size();i++)
        {
            while(!st.empty() && temperatures[st.top()] < temperatures[i] )
            {
                res[st.top()] = i-st.top();
                st.pop();
            }
            st.push(i);
        }
        return res;
    }
};
```

## 621 任务调度器

tricky

```cpp
class Solution {
public:
    static bool cmp(int x, int y){
        return x>y;
    }
    int leastInterval(vector<char>& tasks, int n) {
        int len = tasks.size();
        vector<int> vec(26);
        for(char c: tasks) vec[c-'A']++;
        sort(vec.begin(), vec.end(), cmp);
        int cnt = 0;
        while(cnt<vec.size() && vec[cnt]==vec[0]) cnt++;
        return max(len, cnt+(n+1)*(vec[0]-1));
    }
};
```

## 1 两数之和

[https://leetcode.cn/problems/two-sum/](https://leetcode.cn/problems/two-sum/)

用 unordered_map 储存

```cpp
class Solution {
public:
    vector<int> twoSum(vector<int>& nums, int target) {
        unordered_map<int, int> dict;
        vector<int> res;
        if(nums.size() == 0){
            return res;
        }
        for(int i=0;i<nums.size();i++){
            if(dict.count(nums[i])){
                res.push_back(i);
                res.push_back(dict[nums[i]]);
                return res;
            }
            dict[target-nums[i]] = i;
        }
        return res;
    }
};
```

## 2 两数相交

https://leetcode.cn/problems/intersection-of-two-linked-lists/

链表，注意进位的最后一次运算

```cpp
class Solution {
public:
    ListNode* addTwoNumbers(ListNode* l1, ListNode* l2) {
        //
        // 9 9 9 9 9 9 9
        // 9 9 9 9
        //
        auto dummy = new ListNode(0);
        auto cur = dummy;
        int val;
        int x = 0;
        while(l1 && l2){
            val = l1->val+l2->val + x;
            x = val/10;
            cur->next = new ListNode(val%10);
            cur = cur->next;
            l1 = l1->next;
            l2 = l2->next;
        }
        if(l1==nullptr){
            swap(l1, l2);
        }
        while(l1){
            val = l1->val + x;
            x = val/10;
            cur->next = new ListNode(val%10);
            cur = cur->next;
            l1 = l1->next;
        }
        if(x){
            cur->next = new ListNode(x);
            cur = cur->next;
        }
        return dummy->next;
    }
};
```

## 3 无重复字符的最长子串

[https://leetcode.cn/problems/longest-substring-without-repeating-characters/](https://leetcode.cn/problems/longest-substring-without-repeating-characters/)

双指针 or 滑动窗口？

利用哈希表 unordered_map<char, int>记录字符出现的位置，左指针初始为-1，左开右比，每次更新左指针的最大值

```cpp
class Solution {
public:
    int lengthOfLongestSubstring(string s) {
        int res = 0;
        unordered_map<char, int> cache;
        int left = -1;
        for(int right=0;right<s.size();right++){
            char cur = s[right];
            if(cache.count(cur)){
                left = max(left, cache[cur]);
            }
            res = max(res, right-left);
            cache[cur] = right;
        }
        return res;
    }
};
```

## 4 寻找两个正序数组的中位数

将问题化简，当数组是奇数个时，求中间的那个，偶数时求中间的两位数，为了防止分类讨论，将问题转换为求第(n+m+1)/2 个数字和第(n+m+2)/2 个数字

这道题让我们求两个有序数组的中位数，而且限制了时间复杂度为 O(log (m+n))，看到这个时间复杂度，自然而然的想到了应该使用二分查找法来求解。那么回顾一下中位数的定义，如果某个有序数组长度是奇数，那么其中位数就是最中间那个，如果是偶数，那么就是最中间两个数字的平均值。这里对于两个有序数组也是一样的，假设两个有序数组的长度分别为 m 和 n，由于两个数组长度之和 m+n 的奇偶不确定，因此需要分情况来讨论，对于奇数的情况，直接找到最中间的数即可，偶数的话需要求最中间两个数的平均值。为了简化代码，不分情况讨论，我们使用一个小 trick，我们分别找第 (m+n+1) / 2 个，和 (m+n+2) / 2 个，然后求其平均值即可，这对奇偶数均适用。加入 m+n 为奇数的话，那么其实 (m+n+1) / 2 和 (m+n+2) / 2 的值相等，相当于两个相同的数字相加再除以 2，还是其本身。

这里我们需要定义一个函数来在两个有序数组中找到第 K 个元素，下面重点来看如何实现找到第 K 个元素。首先，为了避免产生新的数组从而增加时间复杂度，我们使用两个变量 i 和 j 分别来标记数组 nums1 和 nums2 的起始位置。然后来处理一些边界问题，比如当某一个数组的起始位置大于等于其数组长度时，说明其所有数字均已经被淘汰了，相当于一个空数组了，那么实际上就变成了在另一个数组中找数字，直接就可以找出来了。还有就是如果 K=1 的话，那么我们只要比较 nums1 和 nums2 的起始位置 i 和 j 上的数字就可以了。难点就在于一般的情况怎么处理？因为我们需要在两个有序数组中找到第 K 个元素，为了加快搜索的速度，我们要使用二分法，对 K 二分，意思是我们需要分别在 nums1 和 nums2 中查找第 K/2 个元素，注意这里由于两个数组的长度不定，所以有可能某个数组没有第 K/2 个数字，所以我们需要先检查一下，数组中到底存不存在第 K/2 个数字，如果存在就取出来，否则就赋值上一个整型最大值。如果某个数组没有第 K/2 个数字，那么我们就淘汰另一个数字的前 K/2 个数字即可。有没有可能两个数组都不存在第 K/2 个数字呢，这道题里是不可能的，因为我们的 K 不是任意给的，而是给的 m+n 的中间值，所以必定至少会有一个数组是存在第 K/2 个数字的。最后就是二分法的核心啦，比较这两个数组的第 K/2 小的数字 midVal1 和 midVal2 的大小，如果第一个数组的第 K/2 个数字小的话，那么说明我们要找的数字肯定不在 nums1 中的前 K/2 个数字，所以我们可以将其淘汰，将 nums1 的起始位置向后移动 K/2 个，并且此时的 K 也自减去 K/2，调用递归。反之，我们淘汰 nums2 中的前 K/2 个数字，并将 nums2 的起始位置向后移动 K/2 个，并且此时的 K 也自减去 K/2，调用递归即可。

```cpp
class Solution {
public:

    double findKth(vector<int>& nums1, int i1, vector<int>& nums2, int i2, int k){
        if(i1 >= nums1.size()) return nums2[i2+k-1];
        if(i2 >= nums2.size()) return nums1[i1+k-1];
        if(k==1) return min(nums1[i1], nums2[i2]);
        int nums1_mid = i1+(k/2)-1 >= nums1.size() ? INT_MAX: nums1[i1+(k/2)-1];
        int nums2_mid = i2+(k/2)-1 >= nums2.size() ? INT_MAX: nums2[i2+(k/2)-1];
        if(nums1_mid > nums2_mid){
            return findKth(nums1, i1, nums2, i2+(k/2), k-k/2);
        }
        return findKth(nums1, i1+(k/2), nums2, i2, k-k/2);
    }
    double findMedianSortedArrays(vector<int>& nums1, vector<int>& nums2) {
        int n = nums1.size();
        int m = nums2.size();
        int left = (m+n+1)/2;
        int right = (m+n+2)/2;
        return (1.0*findKth(nums1, 0, nums2, 0, left) + findKth(nums1, 0, nums2, 0, right))/2;
    }

};
```

## 5 最长回文字串

trick 中间增长法

```jsx
class Solution {
private:
    string res;
public:
    void extend_mid(string &s , int i1, int i2){
        while(i1>=0 && i2<s.size() && s[i1] == s[i2]){
            int len = i2-i1+1;
            if(len>res.size()){
                res = s.substr(i1, len);
            }
            i1--;
            i2++;
        }
    }
    string longestPalindrome(string s) {
        for(int i=0;i<s.size();i++)
        {
            extend_mid(s, i, i+1);
            extend_mid(s, i, i);
        }
        return res;
    }
};
```

## 11 盛最多水的容器

双指针

[https://leetcode.cn/problems/container-with-most-water/](https://leetcode.cn/problems/container-with-most-water/)

left=0， right = size()-1, 当前的水容量是

$$
min(height[left], height[right])* (right-left)
$$

当减少宽度时，如果移动更高的一边，那么容量肯定只会减少，如果移动更高的一边，容量可能减少也可能增加

```cpp
class Solution {
public:
    int maxArea(vector<int>& height) {
        int res = INT_MIN;
        int left = 0;
        int right = height.size()-1;
        while(left<right)
        {
            res = max(res, (right-left)*min(height[left], height[right]));
            if(height[left]<height[right]){
                left++;
            }else{
                right--;
            }
        }
        return res;
    }
};
```

## 15 三数之和

https://leetcode.cn/problems/3sum/

首先对数组进行排序，然后转换为两数之和。（其中要进行去重 both in main and two sum）

```cpp
class Solution {
private:
    vector<vector<int>> res;
public:
    void twoSum(vector<int> nums, int start, int target){
        int left = start+1;
        int right = nums.size()-1;
        while(left<right){
            int cur = nums[left] + nums[right];
            if(cur == target){
                res.push_back(vector<int>{nums[left], nums[right], nums[start]});
                int left_val = nums[left];
                int right_val  = nums[right];
                while(left<right && nums[left] == left_val) left++;
                while(left<right && nums[right] == right_val) right--;
            }else if(cur<target){
                left++;
            }else{
                right--;
            }
        }
    }
    vector<vector<int>> threeSum(vector<int>& nums) {
        sort(nums.begin(), nums.end());
        for(int i=0;i<nums.size();i++)
        {
            if(i!=0 && nums[i] == nums[i-1]) continue;
            twoSum(nums, i, -nums[i]);
        }
        return res;
    }
};
```

## 17 电话号码的字母组合

[https://leetcode.cn/problems/letter-combinations-of-a-phone-number/](https://leetcode.cn/problems/letter-combinations-of-a-phone-number/?favorite=2cktkvj)

```cpp
class Solution {
private:
    unordered_map<char, string> dict = {
        {'2', "abc"},
        {'3', "def"},
        {'4', "ghi"},
        {'5', "jkl"},
        {'6', "mno"},
        {'7', "qprs"},
        {'8', "tuv"},
        {'9', "wxyz"},
    };
    vector<string> res;
public:
    void dfs(string& digits, int idx, string& cur)
    {
        if(idx == digits.size()){
            res.push_back(cur);
        }
        for(int i=0;i<dict[digits[idx]].size();i++)
        {
            cur.push_back(dict[digits[idx]][i]);
            dfs(digits, idx+1, cur);
            cur.pop_back();
        }
    }
    vector<string> letterCombinations(string digits) {
        if(digits.size() == 0) return res;
        string cur = "";
        dfs(digits, 0, cur);
        return res;
    }
};
```

## 19 删除链表的倒数第 N 个结点

[https://leetcode.cn/problems/remove-nth-node-from-end-of-list](https://leetcode.cn/problems/remove-nth-node-from-end-of-list/?favorite=2cktkvj)

```cpp
class Solution {
public:
    ListNode* removeNthFromEnd(ListNode* head, int n) {
        auto dummy = new ListNode(0);
        dummy->next = head;
        auto fast = dummy;
        auto slow = dummy;
        for(int i=0;i<n;i++){
            fast = fast->next;
        }
       while(fast->next){
           fast = fast->next;
           slow = slow->next;
       }
       slow->next = slow->next->next;
       return dummy->next;
    }
};
```

## 20 有效括号

[https://leetcode.cn/problems/valid-parentheses](https://leetcode.cn/problems/valid-parentheses)

使用栈

```cpp
class Solution {
public:
    bool is_pair(char a, char b){
        if(a==')' && b=='(') return true;
        if(a==']' && b=='[') return true;
        if(a=='}' && b=='{') return true;
        return false;
    }
    bool isValid(string s) {
        stack<char> st;
        for(auto c:s){
            if(!st.empty() && is_pair(c, st.top())==true){
                st.pop();
            }else{
                st.push(c);
            }
        }
        return st.empty();
    }
};
```

## 合并两个有序链表

[https://leetcode.cn/problems/merge-two-sorted-lists/](https://leetcode.cn/problems/merge-two-sorted-lists/?favorite=2cktkvj)

1.  递归

```cpp
class Solution {
public:
    ListNode* mergeTwoLists(ListNode* list1, ListNode* list2) {
        if(list1 == nullptr) return list2;
        if(list2 == nullptr) return list1;
        if(list1->val > list2->val){
            swap(list1, list2);
        }
        list1->next = mergeTwoLists(list1->next, list2);
        return list1;
    }
};
```

1. 迭代

```cpp
class Solution {
public:
    ListNode* mergeTwoLists(ListNode* l1, ListNode* l2) {
        if(!l1) return l2;
        if(!l2) return l1;
        ListNode* dummy = new ListNode(0);
        ListNode* tp = dummy;
        while(l1&&l2){
            if(l1->val < l2->val){
                tp->next = l1;
                l1 = l1->next;
            }else{
                tp->next = l2;
                l2 = l2->next;
            }
            tp = tp->next;
        }
        if(l1) tp->next = l1;
        if(l2) tp->next = l2;
        return dummy->next;
    }
};
```

## 括号生成

[https://leetcode.cn/problems/generate-parentheses](https://leetcode.cn/problems/generate-parentheses)

深度搜索，其中注意左括号数量永远大于等于右括号

```cpp
class Solution {
private:
    vector<string> res;
    int n;
public:
    void dfs(string& cur, int left, int right){
        if(left>n || right>n || left<right){
            return;
        }
        if(left==n && right==n){
            res.push_back(cur);
        }

        cur.push_back('(');
        dfs(cur, left+1, right);
        cur.pop_back();

        cur.push_back(')');
        dfs(cur, left, right+1);;
        cur.pop_back();
    }
    vector<string> generateParenthesis(int n) {
        string tp = "";
        this->n = n;
        dfs(tp, 0, 0);
        return res;
    }
};
```

## 23 合并 K 个升序链表

利用堆，注意大于是小根堆，小于是大根堆

```cpp
class Solution {
public:
    class cmp{
        public:
            bool operator ()(ListNode* a, ListNode* b){
                return a->val > b->val;
            }
    };
    ListNode* mergeKLists(vector<ListNode*>& lists) {
        auto dummy = new ListNode(0);
        auto cur = dummy;
        priority_queue<ListNode*, vector<ListNode*>, cmp> q;
        for(auto& node: lists){
            if(node!=nullptr){
                q.push(node);
            }
        }
        while(!q.empty())
        {
            auto front = q.top();
            q.pop();
            cur->next = front;
            cur = cur->next;
            if(front->next != nullptr){
                q.push(front->next);
            }
        }
        return dummy->next;
    }
};
```

## !31 下一个排列

```cpp
class Solution {
public:

    void nextPermutation(vector<int>& nums) {
        int i = nums.size()-1;
        while(i>0){
            if(nums[i-1] < nums[i]){
                break;
            }
            i--;
        }
        if(i==0){
            sort(nums.begin(), nums.end());
            return;
        }
        i = i-1;
        int j = nums.size() - 1;
        while(j>=0){
            if(nums[j] > nums[i]){
                break;
            }
            j--;
        }

        swap(nums[i], nums[j]);
        sort(nums.begin()+i+1, nums.end());
    }
};

```

## 32. 最长有效括号

[https://leetcode.cn/problems/longest-valid-parentheses/](https://leetcode.cn/problems/longest-valid-parentheses/)

动态规划

dp[i] 是 i 个位置的字符为结尾的有效括号长度

当 s[i]是`(`的时候，dp[i]为 0，

当 s[i]是`)`的时候, 讨论 s[i-1]的情况

当 s[i-1]是`(`，那么 dp[i] = dp[i-2]+2; 注意 i-2 的是合法

当 s[i-2]是`)`, 那么需要讨论 i-1-len，len 是 dp[i-1]的值，如果 s[i-1-len]是`(`，那么 dp[i] = dp[i-1]+dp[i-1-dp[i-1]-1] +2

最终返回所有 dp 的最大值

```cpp
class Solution {
public:
    int longestValidParentheses(string s) {
        vector<int> dp(s.size(), 0);
        if(s.size() == 0) return 0;
        int res = 0;
        for(int i=1;i<s.size();i++)
        {
            if(s[i] == '('){
                dp[i] = 0;
            }else if(s[i] == ')'){
                if(s[i-1] == '('){
                    dp[i] = 2;
                    if(i-2>=0){
                        dp[i] += dp[i-2];
                    }
                }else{
                    if(dp[i-1]>0){
                        int len = dp[i-1];
                        int start = i-1-len;
                        if(start>=0){
                            if(s[start] == '('){
                                dp[i] = dp[i-1] + 2;
                                if(start-1 >=0){
                                    dp[i] += dp[start-1];
                                }
                            }
                        }else{
                            dp[i] = 0;
                        }
                    }
                }
            }
            res = max(res, dp[i]);
        }
        return res;
    }
};
```

## 49 [字母异分位词](https://leetcode.cn/problems/group-anagrams/)

利用 dict

```cpp
class Solution {
private:
    vector<vector<string>> res;
    unordered_map<string, vector<string>> cache;
    vector<bool> visit;
public:
    vector<vector<string>> groupAnagrams(vector<string>& strs) {
        for(auto& str: strs){
            auto reve = str;
            sort(reve.begin(), reve.end());
            cache[reve].push_back(str);
        }
        for(auto it=cache.begin();it!=cache.end();it++){
            res.push_back(it->second);
        }
        return res;
    }
};
```

## 53 最大子数组和

动态规划

```cpp
class Solution {
public:
    int maxSubArray(vector<int>& nums) {
         int cur = nums[0];
         int res = nums[0];
         for(int i=1;i<nums.size();i++){
             cur = max(nums[i], nums[i]+cur);
             res = max(res, cur);
         }
         return res;
    }
};
```

## 55 跳跃游戏

[https://leetcode.cn/problems/jump-game/](https://leetcode.cn/problems/jump-game/)

贪心，定义一个最远能跳的距离 most_right, 遍历数组，如果能小于等于 most_right,那么通过该下表推进 most_right

```cpp
class Solution {

public:
    bool canJump(vector<int>& nums) {
       int most_right = 0;
       for(int i=0;i<nums.size();i++)
       {
           if(i <= most_right){
               most_right = max(most_right, i+nums[i]);
               if(most_right>=nums.size()-1){
                   return true;
               }
           }
       }
       return false;
    }
};
```

## 33 \***\*[搜索旋转排序数组](https://leetcode.cn/problems/search-in-rotated-sorted-array/)\*\***

二分搜索，分类讨论，target 是在两块区间的那个部分（和 nums[0]作比较），然后再讨论 num[mid]在哪个区间

```cpp
class Solution {
public:
    int search(vector<int>& nums, int target) {
        int left = 0;
        int right = nums.size();
        while(left<right){
            int mid = left + (right-left)/2;
            if(nums[mid] == target) return mid;
            if(target==nums[0]) return 0;
            if(target>nums[0])
            {
                if(nums[mid] > target){
                    right = mid;
                }else{
                    // target < mid;
                    if(nums[mid] > nums[0]){
                        left = mid+1;
                    }else{
                        right = mid;
                    }
                }
            }else{
                if(nums[mid]<target){
                    left = mid+1;
                }else{
                    if(nums[mid]>nums[0]){
                        left = mid+1;
                    }else{
                        right = mid;
                    }
                }
            }
        }
        return -1;
    }
};
```

## 56 合并区间

[https://leetcode.cn/problems/merge-intervals](https://leetcode.cn/problems/merge-intervals)

首先对区间进行 sort，第一排序顺序是左边界，第二排序顺序号是右边边界，建立 result vector 数组，对区间进行遍历，如果区间右边界大于 result 数组最后一个的右边界，那么 push 进去，否则对 result 最后一个的右边界进行扩增

```cpp
class Solution {
public:
    static bool cmp(vector<int> a, vector<int> b){
        if(a[0] != b[0]) return a[0] < b[0];
        return a[1] < b[1];
    }
    vector<vector<int>> merge(vector<vector<int>>& intervals) {
        sort(intervals.begin(), intervals.end(), cmp);
        vector<vector<int>> res;
        res.push_back(intervals[0]);
        for(int i=1;i<intervals.size();i++)
        {
            auto cur = intervals[i];
            auto last = res[res.size() - 1];
            if(cur[0] > last[1]){
                res.push_back(cur);
            }else{
                res[res.size()-1][1] = max(last[1], cur[1]);
            }
        }
        return res;
    }
};
```

## 34 \***\*[在排序数组中查找元素的第一个和最后一个位置](https://leetcode.cn/problems/find-first-and-last-position-of-element-in-sorted-array/)\*\***

[https://leetcode.cn/problems/find-first-and-last-position-of-element-in-sorted-array](https://leetcode.cn/problems/find-first-and-last-position-of-element-in-sorted-array/)

upper_bound 和 lower_bound 函数的实现

要注意两点，第一个是注意返回的 idx 的合法性，第二个是 upper_bound 返回的 index 需要-1

```cpp
class Solution {
public:
    int lower_bound(vector<int>& nums, int target){
        int left = 0;
        int right = nums.size();
        while(left<right)
        {
            int mid = left + (right-left)/2;
            if(target>nums[mid]){
                // (mid, right)
                left = mid+1;
            }else if(target==nums[mid]){
                right = mid;
            }else{
                 //
                 // target < nums[mid]
                 right = mid;
            }
        }
        return left;
    }
    int upper_bound(vector<int>& nums, int target){
        int left = 0;
        int right = nums.size();
        while(left < right){
            int mid = left+(right-left)/2;
            if(target>nums[mid]){
                // ()
                left = mid+1;
            }else if(target == nums[mid]){
                left = mid+1;
            }else{
                // target < nums[mid]
                right = mid;
            }
        }
        return left;
    }
    vector<int> searchRange(vector<int>& nums, int target) {
        int upper = upper_bound(nums, target);
        int lower = lower_bound(nums, target);
        vector<int> res{-1, -1};
        if(lower >=0 && lower<nums.size() && nums[lower] == target) res[0] = lower;
        if(upper-1 >= 0 && upper-1<nums.size() && nums[upper-1] == target) res[1] = upper-1;
        return res;
    }
};
```

## ！39 组合总和

[https://leetcode.cn/problems/combination-sum/submissions/](https://leetcode.cn/problems/combination-sum/submissions/)
经典题目，回溯剪枝，当前节点 选择还是不选择？ 选择的话：

```cpp
cur_vec.push_back(candidates[cur_idx]);
dfs(candidates, cur_val+candidates[cur_idx], cur_idx, target, cur_vec);
cur_vec.pop_back();
```

如果不选择

```cpp
dfs(candidates, cur_val, cur_idx+1, target, cur_vec);
```

```cpp
class Solution {
private:
    vector<vector<int>> res;
public:
    void dfs(vector<int>& candidates, int cur_val, int cur_idx, int target, vector<int>& cur_vec){
        if(cur_val == target){
            res.push_back(cur_vec);
            return;
        }
        if(cur_val > target || cur_idx>=candidates.size()){
            return;
        }
        // 选择当前路径
        cur_vec.push_back(candidates[cur_idx]);
        dfs(candidates, cur_val+candidates[cur_idx], cur_idx, target, cur_vec);
        cur_vec.pop_back();
        // 不选择当前路径
        dfs(candidates, cur_val, cur_idx+1, target, cur_vec);

    }
    vector<vector<int>> combinationSum(vector<int>& candidates, int target) {
        vector<int> cur_vec;
        dfs(candidates, 0, 0, target, cur_vec);
        return res;
    }
};
```

## 42 接雨水

维护一个 left_max 和 right_max

```cpp

class Solution {
public:
    int trap(vector<int>& height) {
        int left = 0;
        int right = height.size()-1;
        int res = 0;
        int max_left = height[left];
        int max_right = height[right];
        while(left < right)
        {
            if(height[left] > height[right]){
                    res += max(0, max_right - height[right]);
                    max_right = max(max_right, height[right]);
                    right--;
            }else{
                    res += max(0, max_left-height[left]);
                    max_left = max(max_left, height[left]);
                    left++;
            }
        }
        return res;
    }
};
```

## 46 全排列

[https://leetcode.cn/problems/permutations](https://leetcode.cn/problems/permutations)

回溯，visit 数组

```cpp
class Solution {
private:
    vector<vector<int>> res;
    vector<bool> visit;
public:
    void dfs(vector<int> & nums, vector<int>& cur_vec){
        if(cur_vec.size() == nums.size()){
            res.push_back(cur_vec);
            return;
        }
        for(int i=0;i<nums.size();i++)
        {
            if(visit[i] == false)
            {
                visit[i] = true;
                cur_vec.push_back(nums[i]);
                dfs(nums, cur_vec);
                cur_vec.pop_back();
                visit[i] = false;
            }
        }
    }
    vector<vector<int>> permute(vector<int>& nums) {
        visit = vector<bool> (nums.size(), false);
        vector<int> cur_vec;

        dfs(nums, cur_vec);

        return res;
    }
};
```

## 48 旋转图像

[https://leetcode.cn/problems/rotate-image/](https://leetcode.cn/problems/rotate-image/)

先进行轴翻转，然后行 reverse，在轴翻转的过程中注意 i 的范围是(0, n), j 的范围是(i, m)，否则的话翻过来再翻过去等于没翻

```cpp
class Solution {
public:
    void rotate(vector<vector<int>>& matrix) {
        int n = matrix.size();
        int m = matrix[0].size();
        for(int i=0;i<n;i++)
        {
            for(int j=i;j<m;j++)
            {
                swap(matrix[i][j], matrix[j][i]);
            }
        }
        for(auto& row: matrix)
        {
            reverse(row.begin(), row.end());
        }
    }
};
```

## 53 最大子数组和

[https://leetcode.cn/problems/maximum-subarray/submissions/](https://leetcode.cn/problems/maximum-subarray/submissions/)

动态规划

```cpp
class Solution {
public:
    int maxSubArray(vector<int>& nums) {
         int ans = nums[0];
         int cur = nums[0];
         for(int i=1;i<nums.size();i++){
             cur = max(nums[i], cur+nums[i]);
             ans = max(ans, cur);
         }
         return ans;
    }
};
```

## 49 分母异位词

[https://leetcode.cn/problems/group-anagrams/](https://leetcode.cn/problems/group-anagrams/)?

利用 dict 和 sort 函数

```cpp
class Solution {
private:
    vector<vector<string>> res;
    unordered_map<string, vector<string>> cache;
    vector<bool> visit;
public:
    vector<vector<string>> groupAnagrams(vector<string>& strs) {
        for(auto& str: strs){
            auto reve = str;
            sort(reve.begin(), reve.end());
            cache[reve].push_back(str);
        }
        for(auto it=cache.begin();it!=cache.end();it++){
            res.push_back(it->second);
        }
        return res;
    }
};
```

## 不同路径

动态规划

```cpp

class Solution {
public:
    int uniquePaths(int m, int n) {
        swap(n, m);
        vector<vector<int>> f(n, vector<int>(m, 0));
        for(int i=0;i<n;i++){
            f[i][0] = 1;
        }
        for(int i=0;i<m;i++){
            f[0][i] = 1;
        }
        for(int i=1;i<n;i++){
            for(int j=1;j<m;j++){
                f[i][j] = f[i-1][j]+f[i][j-1];
            }
        }
        return f[0][0];
    }
};
```

## 64 最小路径和

动态规划

[https://leetcode.cn/problems/minimum-path-sum/](https://leetcode.cn/problems/minimum-path-sum/)

```cpp
class Solution {
public:
    int minPathSum(vector<vector<int>>& grid) {
        int n = grid.size();
        int m = grid[0].size();
        vector<vector<int>> f(n, vector<int>(m, 0));
        f[0][0] = grid[0][0];
        for(int i=1;i<n;i++){
            f[i][0] = f[i-1][0] + grid[i][0];
        }
        for(int i=1;i<m;i++){
            f[0][i] = f[0][i-1] + grid[0][i];
        }
        for(int i=1;i<n;i++)
        {
            for(int j=1;j<m;j++)
            {
                f[i][j] = min(f[i-1][j], f[i][j-1]) + grid[i][j];
            }
        }
        return f[n-1][m-1];
    }
};
```

## 70 爬楼梯

动态规划

[https://leetcode.cn/problems/climbing-stairs/](https://leetcode.cn/problems/climbing-stairs/)

```cpp
class Solution {
public:
    int climbStairs(int n) {
        int prev = 1;
        int cur = 2;
        if(n==1) return 1;
        if(n==2) return 2;
        for(int i=3;i<=n;i++){
            int tp = cur;
            cur = prev+cur;
            prev = tp;
        }
        return cur;
    }
};
```

## 72 编辑距离

动态规划

```cpp
class Solution {
private:
    vector<vector<int>> f;
public:
    int dfs(string& word1, string& word2, int i, int j){
        if(i==0){
            return j;
        }
        if(j==0){
            return i;
        }
        if(word1[i-1] == word2[j-1]){
            if(f[i-1][j-1] == -1){
                f[i][j] = dfs(word1, word2, i-1, j-1);
            }else{
                f[i][j] = f[i-1][j-1];
            }
            return f[i][j];
        }
        int delete_ , insert_, replace_;
        delete_ =  f[i-1][j] != -1 ? f[i-1][j]+1 :  dfs(word1, word2, i-1, j)+1;
        insert_ =  f[i][j-1] != -1  ? f[i][j-1]+1 :  dfs(word1, word2, i, j-1)+1;
        replace_ = f[i-1][j-1] != -1  ? f[i-1][j-1]+1 : dfs(word1, word2, i-1, j-1)+1;
        f[i][j] = min(min(delete_, insert_), replace_);
        return f[i][j];
    }
    int minDistance(string word1, string word2) {
        f = vector<vector<int>> (word1.size()+1, vector<int>(word2.size()+1, -1));
        return dfs(word1, word2, word1.size(), word2.size());
    }
};
```

# 75 颜色分类

[https://leetcode.cn/problems/sort-colors/](https://leetcode.cn/problems/sort-colors/)

三指针

本质就是三个指针，头指针和中指针负责 0 和 1 的交换，中指针和尾指针负责把 2 移到末

```cpp
class Solution {
public:
    void sortColors(vector<int>& nums) {
        // 三指针
        int red_idx = 0;
        int blue_idx = nums.size()-1;
        int white_idx = 0;
        while(white_idx <= blue_idx){
           if(nums[white_idx] == 1){
               white_idx++;
           }else if(nums[white_idx] == 0){
               swap(nums[white_idx], nums[red_idx]);
               red_idx++;
               white_idx++;
           }else{
               swap(nums[blue_idx], nums[white_idx]);
               blue_idx--;
           }
        }
    }
};
```

## 78 子集

回溯， 两条路，选或者不选

```cpp
class Solution {
private:
    vector<vector<int>> res;
public:
    void dfs(vector<int>& nums, vector<int>& cur, int idx){
        if(idx>=nums.size()){
            res.push_back(cur);
            return;
        }
        dfs(nums, cur, idx+1);
        cur.push_back(nums[idx]);
        dfs(nums, cur, idx+1);
        cur.pop_back();
    }
    vector<vector<int>> subsets(vector<int>& nums) {
        vector<int> cur;
        dfs(nums, cur, 0);
        return res;
    }
};
```

## 柱状图中的最大矩形

[https://leetcode.cn/problems/maximal-rectangle/](https://leetcode.cn/problems/maximal-rectangle/)

单调栈

```cpp
class Solution {
public:
    int largestRectangleArea(vector<int>& heights) {
        vector<int> left_min(heights.size(), -1);
        vector<int> right_min(heights.size(), heights.size());
        stack<int> stk;
        for(int i=0;i<heights.size();i++)
        {
            while(!stk.empty() && heights[i] < heights[stk.top()]){
                right_min[stk.top()] = i;
                stk.pop();
            }
            stk.push(i);
        }
        stk = stack<int>{};
        for(int i=heights.size()-1;i>=0;i--)
        {
            while(!stk.empty() && heights[i] < heights[stk.top()]){
                left_min[stk.top()] = i;
                stk.pop();
            }
            stk.push(i);
        }
        int res = 0;

        for(int i=0;i<heights.size();i++)
        {
            int left = left_min[i];
            int right = right_min[i];
            res = max(res, (right-left-1)* heights[i]);
        }
        return res;
    }

};
```

## 85 最大矩形

单调栈

！ 注意 matrix 是字符的情况，转换成 int 的 matrix 的时候一定要注意先决条件 matrix[i][j] == '0'

```cpp
class Solution {

public:
    int maximalRectangle(vector<vector<char>>& matrix) {
        int n = matrix.size();
        int m = matrix[0].size();
        int res = 0;
        vector<vector<int>> int_matrix(n, vector<int>(m, 0));
        for(int i=0;i<n;i++)
        {
            int_matrix[i][0] = matrix[i][0] == '0' ? 0 : 1;
        }

        for(int i=0;i<n;i++)
        {
            for(int j=1;j<m;j++)
            {
                if(matrix[i][j] == '1')
                int_matrix[i][j] = int_matrix[i][j-1] + 1;
            }
        }
        for(int j=0;j<m;j++)
        {
            vector<int> down_min(n, n);
            vector<int> up_min(n, -1);
            stack<int> stk;
            for(int i=0;i<n;i++)
            {
                while(!stk.empty() && int_matrix[stk.top()][j] > int_matrix[i][j]){
                    down_min[stk.top()] = i;
                    stk.pop();
                }
                stk.push(i);
            }
            stk = stack<int>{};
            for(int i=n-1;i>=0;i--)
            {
                while(!stk.empty() && int_matrix[stk.top()][j] > int_matrix[i][j]){
                    up_min[stk.top()] = i;
                    stk.pop();
                }
                stk.push(i);
            }
            for(int i=0;i<n;i++)
            {
                res = max(res, int_matrix[i][j] * (down_min[i]-up_min[i]-1));
            }
        }
        return res;
    }
};
```

## 94 二叉树的中序遍历

掌握非递归方法

```cpp
class Solution {
public:
    vector<int> inorderTraversal(TreeNode* root) {
        stack<TreeNode* > stk;
        vector<int> res;
        auto cur = root;
        while(cur || !stk.empty())
        {
            while(cur){
                stk.push(cur);
                cur = cur->left;
            }
            auto top = stk.top();
            stk.pop();
            res.push_back(top->val);
            if(top->right)
            cur = top->right;
        }
        return res;
    }
};
```

## 不同的二叉搜索树

[https://leetcode.cn/problems/unique-binary-search-trees](https://leetcode.cn/problems/unique-binary-search-trees)

```cpp
class Solution {
public:
    int numTrees(int n) {
        vector<int> g(n+1);
        if(n == 1) return 1;
        g[1] = 1;
        g[2] = 2;
        g[0] = 1;
        for(int i=3;i<=n;i++)
        {
            for(int j=1;j<=i;j++)
            {
                g[i] += g[j-1] * g[i-j];
            }
        }
        return g[n];
    }
};
```

## 98 验证二叉搜索树

[https://leetcode.cn/problems/validate-binary-search-tree/](https://leetcode.cn/problems/validate-binary-search-tree/)

利用二叉搜索树中序遍历的性质

```cpp
class Solution {
private:
    bool res = true;
    int cur = INT_MIN;
    bool init = false;
public:
    void dfs(TreeNode* root){
        if(root==nullptr) return;
        dfs(root->left);
        if(init == false){
            init = true;

        }else{
            if(root->val <= cur){
                res = false;
            }
        }
        cur = root->val;
        dfs(root->right);
    }
    bool isValidBST(TreeNode* root) {
        dfs(root);
        return res;
    }
};
```

## 101 对称二叉树

[https://leetcode.cn/problems/symmetric-tree](https://leetcode.cn/problems/symmetric-tree) 递归

```cpp
class Solution {
public:
    bool dfs(TreeNode* left, TreeNode* right){
        if(left==nullptr && right == nullptr) return true;
        if(left==nullptr || right == nullptr) return false;
        if(left->val != right->val) return false;
        return dfs(left->right, right->left) && dfs(left->left, right->right);
    }
    bool isSymmetric(TreeNode* root) {
        if(root==nullptr) return true;
        return dfs(root->left, root->right);
    }
};
```

## 102 二叉树的层序遍历

[https://leetcode.cn/problems/binary-tree-level-order-traversal](https://leetcode.cn/problems/binary-tree-level-order-traversal)

队列

```cpp
class Solution {
public:
    vector<vector<int>> levelOrder(TreeNode* root) {
        queue<TreeNode*> q;
        q.push(root);
        vector<vector<int>> res;
        if(root==nullptr) return res;
        while(!q.empty())
        {
            int sz = q.size();
            vector<int> cur;
            while(sz--){
                auto top = q.front();
                q.pop();
                cur.push_back(top->val);
                if(top->left){
                    q.push(top->left);
                }
                if(top->right){
                    q.push(top->right);
                }
            }
            res.push_back(cur);
        }
        return res;
    }
};
```

## 104 二叉树的最大深度

[https://leetcode.cn/problems/maximum-depth-of-binary-tree/](https://leetcode.cn/problems/maximum-depth-of-binary-tree/)

```cpp
class Solution {
public:
    int maxDepth(TreeNode* root) {
        if(root==nullptr) return 0;
        int left = maxDepth(root->left);
        int right = maxDepth(root->right);
        return 1+max(left, right);
    }
};
```

## 105 前序中序构造二叉树

[https://leetcode.cn/problems/construct-binary-tree-from-preorder-and-inorder-traversal/](https://leetcode.cn/problems/construct-binary-tree-from-preorder-and-inorder-traversal/)

```cpp
class Solution {
public:
    TreeNode* build(vector<int>& preorder, int p_l, int p_r, vector<int>& inorder, int i_l, int i_r)
    {
        if(p_l > p_r) return nullptr;
        int root_val = preorder[p_l];
        int i;
        for(i=i_l;i<=i_r;i++)
        {
            if(inorder[i] == root_val){
                break;
            }
        }
        int left_num = i-i_l;
        auto root = new TreeNode(root_val);
        root->left = build(preorder, p_l+1, p_l+left_num, inorder, i_l, i-1);
        root->right = build(preorder, p_l+left_num+1, p_r, inorder, i+1, i_r);
        return root;
    }
    TreeNode* buildTree(vector<int>& preorder, vector<int>& inorder) {
        return build(preorder, 0, preorder.size()-1, inorder, 0, inorder.size()-1);
    }
};
```

## 114 二叉树展开为链表

[https://leetcode.cn/problems/flatten-binary-tree-to-linked-list/](https://leetcode.cn/problems/flatten-binary-tree-to-linked-list/)

递归解决

```cpp
class Solution {
public:
    TreeNode* dfs(TreeNode* root){
        if(root == nullptr){
            return root;
        }
        auto left = dfs(root->left);
        auto right = dfs(root->right);
        if(left == nullptr){
            root->right = right;
            root->left = nullptr;
            return root;
        }
        auto cur = left;
        while(cur->right){
            cur = cur->right;
        }
        root->right = left;
        root->left = nullptr;
        cur->right = right;
        return root;
    }
    void flatten(TreeNode* root) {
        dfs(root);
    }
};
```

## 124 二叉树的最大路径和

[https://leetcode.cn/problems/binary-tree-maximum-path-sum](https://leetcode.cn/problems/binary-tree-maximum-path-sum)

递归

```cpp
class Solution {
private:
    int ans = INT_MIN;
public:
    int dfs(TreeNode* root){
        if(!root) return 0;
        int left = dfs(root->left);
        int right = dfs(root->right);
        int val = root->val + max(left, 0) + max(right, 0);
        ans = max(ans, val);
        return root->val + max(0, max(left, right));
    }
    int maxPathSum(TreeNode* root) {
        dfs(root);
        return ans;
    }
};
```

## 128 最长连续序列

用 map 或者 set（有序的）， 并且用 iterator

```cpp
class Solution {
private:
    set<int> cache;
    int res = 1;
public:
    int longestConsecutive(vector<int>& nums) {
        for(int x: nums){
            cache.insert(x);
        }
        int res = 0;
        auto it =  cache.begin();
        int streak = 1;
        while(it!=cache.end())
        {
            streak = 1;
            while(it!=cache.end() &&cache.count(*it+1)!=0){
                it++;
                streak++;
            }
            res = max(res, streak);
            it++;
        }
        return res;
    }
};
```

## 136 只出现一次的数字

利用亦或

```cpp
class Solution {
public:
    int singleNumber(vector<int>& nums) {
        int x = 0;
        for(int num: nums){
            x = x^num;
        }
        return x;
    }
};
```

## 139 单词拆分

[https://leetcode.cn/problems/word-break](https://leetcode.cn/problems/word-break)

动态规划

```cpp
class Solution {
private:
    vector<bool> dp;
public:
    bool wordBreak(string s, vector<string>& wordDict) {
        dp = vector<bool>(s.size()+1, false);
        dp[0] = true;
        for(int i=1;i<=s.size();i++)
        {
            if(dp[i-1] == true){
                for(string& word: wordDict){
                    bool match = true;
                    for(int j=0;j<word.size();j++)
                    {
                        if(word[j] != s[i-1+j]){
                            match = false;
                            break;
                        }
                    }
                    if(match == true){
                        dp[i+word.size()-1] = true;
                    }
                }
            }
        }
        return dp[s.size()];
    }
};
```

# 剑指 Offer 12. 矩阵中的路径

用 visit 数组

```c++
class Solution {
private:
    vector<vector<bool>> visit;
    int n;
    int m;
    bool res =false;
    vector<vector<int>> directions{{1, 0}, {-1, 0}, {0, 1}, {0, -1}};
public:
    void dfs(vector<vector<char>>& board, string& word, int i, int j, int cur){
        if(i<0 || i>=n || j<0 || j>=m || visit[i][j] || res==true || word[cur]!= board[i][j]){
            return;
        }
        if(cur==word.size()-1){
            res = true;
            return;
        }
        visit[i][j] = true;
        for(auto& direction: directions){
            int next_i = i + direction[0];
            int next_j = j + direction[1];
            dfs(board, word, next_i, next_j, cur+1);
        }
        visit[i][j] = false;
    }
    bool exist(vector<vector<char>>& board, string word) {
        n = board.size();
        if(n==0) return false;
        m = board[0].size();
        visit = vector<vector<bool>>(n, vector<bool>(m, false));
        for(int i=0;i<n && !res; i++)
        {
            for(int j=0;j<m && !res ;j++){
                dfs(board, word, i, j, 0);
            }
        }
        return res;
    }
};
```

# 剑指 Offer 14- I. 剪绳子

动态规划，切出来的部分分为两种情况：
一种继续切割，另一种一整块，不切割

```c++
class Solution {
public:
    int cuttingRope(int n) {
        vector<int> f(1005);
        f[1] = 1;
        for(int i=2;i<=n;i++){
            for(int j=1;j<=i/2;j++){
                int x1 = j;
                int x2 = i-j;
                f[i] = max(f[i], f[x1]*x2);
                f[i] = max(f[i], x1*x2);

            }
        }
        return f[n];
    }
};
```

# 剑指 Offer 16. 数值的整数次方

递归的方式（其实是二分）

```c++
class Solution {
public:
    double fun(double x, int n){
        if(n==0){
            return 1;
        }
        double half = fun(x, n/2);
        return n%2==0? half*half : x*half*half;
    }
    double myPow(double x, int n) {
        long long N = n;
        return n>0 ? fun(x, N) : 1.0/fun(x, -N);
    }
};
```

# 剑指 Offer 26. 树的子结构

递归

```c++
class Solution {

public:
    bool dfs(TreeNode* A, TreeNode* B){
        if(!B) return true;
        if(!A) return false;
        bool left = dfs(A->left, B->left);
        bool right = dfs(A->right, B->right);
        return A->val == B->val && left && right;
    }
    bool isSubStructure(TreeNode* A, TreeNode* B) {
        if(!A || !B) return false;
        return dfs(A, B) || isSubStructure(A->left, B) || isSubStructure(A->right, B);
    }
};
```

# 剑指 Offer 31. 栈的压入、弹出序列

```c++
class Solution {
public:
    bool validateStackSequences(vector<int>& pushed, vector<int>& popped) {
        stack<int> st;
        int idx = 0;
        for(int i=0;i<pushed.size();i++){
            st.push(pushed[i]);
            while(!st.empty() && st.top() == popped[idx]){
                idx++;
                st.pop();
            }
        }
        return st.empty();
    }
};
```

# 剑指 Offer 33. 二叉搜索树的后序遍历序列

遍历，`veriy(vector<int>& postorder, int begin, int end)`， i 从 begin 到 root 是小于， begin+1，到 end 是小于，最后判断 i 是否等于 end。

```c++
class Solution {
public:
    bool veriy(vector<int>& postorder, int begin, int end){
        if(begin >= end) return true;
        int root_val = postorder[end];
        int i = begin;
        while(i<end && postorder[i]<root_val){
            i++;
        }
        int left = i-1;
        while(i<end && postorder[i]>root_val){
            i++;
        }
        return i==end && veriy(postorder, begin, left) && veriy(postorder, left+1, end-1);
    }
    bool verifyPostorder(vector<int>& postorder) {
        return veriy(postorder, 0, postorder.size()-1);
    }
};
```

# 剑指 Offer 34. 二叉树中和为某一值的路径

dfs

```c++
class Solution {
private:
    vector<vector<int>> res;
public:
    void dfs(TreeNode* root, vector<int>& tp, int target){
        if(root==nullptr) return;
        if(target == root->val && root->left==nullptr && root->right==nullptr){
            tp.push_back(root->val);
            res.push_back(tp);
            tp.pop_back();
            return;
        }
        if(root->left){
            tp.push_back(root->val);
            dfs(root->left, tp, target - root->val);
            tp.pop_back();
        }

        if(root->right){
            tp.push_back(root->val);
            dfs(root->right, tp, target - root->val);
            tp.pop_back();
        }

    }
    vector<vector<int>> pathSum(TreeNode* root, int target) {
        vector<int> tp;
        dfs(root, tp, target);
        return res;
    }
};
```

# 剑指 Offer 35. 复杂链表的复制

用一个 unordered_map 存储对应的节点，然后利用后序遍历，当遍历完成之后，map 也做好了

```c++
class Solution {
private:
    unordered_map<Node*, Node*> cache;
public:
    Node* copyRandomList(Node* head) {
        if(head == nullptr) return nullptr;
        auto root = new Node(head->val);
        cache[head] = root;
        root->next = copyRandomList(head->next);
        root->random = cache[head->random];
        return root;
    }
};
```

# 剑指 Offer 36. 二叉搜索树与双向链表

head 和 prev

```c++
class Solution {
private:
    Node* prev = nullptr;
    Node* head = nullptr;
public:
    void inorder(Node* root){
        if(!root) return;
        inorder(root->left);
        if(!head){
            head = root;
        }
        auto right = root->right;
        if(prev){
            prev->right = root;
            root->left = prev;
        }
        prev = root;
        inorder(right);
    }
    Node* treeToDoublyList(Node* root) {
        if(!root) return root;
        inorder(root);
        prev->right = head;
        head->left = prev;
        return head;
    }
};
```

# 剑指 Offer 38. 字符串的排列

1、对字符串进行排序
2、`if(i!=0 && s[i]==s[i-1] && visit[i-1]==true) continue;` 在 dfs 的时候去重

````c++
class Solution {
private:
    Node* prev = nullptr;
    Node* head = nullptr;
public:
    void inorder(Node* root){
        if(!root) return;
        inorder(root->left);
        if(!head){
            head = root;
        }
        auto right = root->right;
        if(prev){
            prev->right = root;
            root->left = prev;
        }
        prev = root;
        inorder(right);
    }
    Node* treeToDoublyList(Node* root) {
        if(!root) return root;
        inorder(root);
        prev->right = head;
        head->left = prev;
        return head;
    }
};
# 剑指 Offer 46. 把数字翻译成字符串
```c++
class Solution {
public:
    int translateNum(int num) {
        string str = to_string(num);
        vector<int> dp(str.size()+1);
        dp[1] = 1;
        dp[0] = 1;
        for(int i=2;i<=str.size();i++){
            int x = (str[i-2] - '0') * 10 + (str[i-1] - '0');
            if(x>=10 && x<=25){
                dp[i] = dp[i-2]+dp[i-1];
            }else{
                dp[i] = dp[i-1];
            }
        }
        return dp[str.size()];
    }
};
````

# 剑指 Offer 46. 把数字翻译成字符串

动态规划
如果`str[i-1]+str[i]` 在 10 到 25 之间的数字

$$
f_{i} = f_{i-1} + f_{i-2}
$$

否则

$$
f_{i} = f_{i-1}
$$

```c++
class Solution {
public:
    int translateNum(int num) {
        string str = to_string(num);
        vector<int> dp(str.size()+1);
        dp[1] = 1;
        dp[0] = 1;
        for(int i=2;i<=str.size();i++){
            int x = (str[i-2] - '0') * 10 + (str[i-1] - '0');
            if(x>=10 && x<=25){
                dp[i] = dp[i-2]+dp[i-1];
            }else{
                dp[i] = dp[i-1];
            }
        }

        return dp[str.size()];
    }
};
```

# 剑指 Offer 47. 礼物的最大价值

动态规划

```c++
class Solution {
public:
    int maxValue(vector<vector<int>>& grid) {
        int n = grid.size();
        int m = grid[0].size();
        vector<vector<int>> dp(n, vector<int>(m, 0));
        dp[0][0] = grid[0][0];
        for(int i=1;i<n;i++)
        {
            dp[i][0] = dp[i-1][0] + grid[i][0];
        }
        for(int j=1;j<m;j++)
        {
            dp[0][j] = dp[0][j-1] + grid[0][j];
        }
        for(int i=1;i<n;i++)
        {
            for(int j=1;j<m;j++){
                dp[i][j] = max(dp[i-1][j], dp[i][j-1]) + grid[i][j];
            }
        }
        return dp[n-1][m-1];
    }
};
```

# 剑指 Offer 49. 丑数

类似于合并三个有序链表

```c++
class Solution {
public:
    int nthUglyNumber(int n) {
        vector<int> f(n, 0);
        f[0] = 1;
        int i2 = 0;
        int i3 = 0;
        int i5 = 0;
        for(int i=1;i<n;i++){
            int x2 = f[i2]*2;
            int x3 = f[i3]*3;
            int x5 = f[i5]*5;
            int min_val = min(min(x2, x3), x5);
            f[i] = min_val;
            if(x2 == min_val){
                i2++;
            }
            if(x3 == min_val){
                i3++;
            }
            if(x5 == min_val){
                i5++;
            }
        }
        return f[n-1];
    }
};
```

# 剑指 Offer 56 - I. 数组中数字出现的次数

首先将所有数字进行异或操作，得到 x， x 中找到那个为 1 的位，根据异或为 1 的那个位将数字分为两组

```c++
class Solution {
public:
    vector<int> singleNumbers(vector<int>& nums) {
        int x = 0;
        for(auto i:nums){
            x = (x^i);
        }
        int d = 1;
        while((d&x) != 0){
            d = d<<1;
        }
        vector<int> ans{0, 0};
        for(int i=0;i<nums.size();i++){
            if((nums[i] & d) == 0){
                ans[0] = ans[0] ^ nums[i];
            }else{
                ans[1] = ans[1] ^ nums[i];
            }
        }
        return ans;
    }
};

```

# 剑指 Offer 56 - II. 数组中数字出现的次数 II

题目大概意思是有一个数只出现过 1 次，其他数都出现过 3 次。
解法是每个位上的 1 进行统计，然后对三取余，如果是 1 的话说明只出现过一次的数这个位上有 1

```c++
class Solution {
public:
    int singleNumber(vector<int>& nums) {
        vector<int> bits(32, 0);
        for(int num: nums){
            int x = 1;
            int i = 0;
            while(num){
                bits[i] += num & x;
                num = num >> 1;
                i++;
            }
        }
        int x = 0;
        int ans = 0;
        for(int i=0;i<32;i++){
            ans += bits[i]%3 * pow(2, x);
            x++;
        }
        return ans;
    }
};
```

# 剑指 Offer 60. n 个骰子的点数

逆向推导

![image-1680526735131](/upload/2023/04/image-1680526735131.png)

# 剑指 Offer 64. 求 1+2+…+n

后序遍历

```c++
class Solution {
private:
    int x = 0;
public:
    int sumNums(int n) {
        int a = n>=1 && sumNums(n-1);
        x += n;
        return x;
    }
};
```

# 剑指 Offer 66. 构建乘积数组

构建从左边和从右边的累计数组

```c++
class Solution {
public:
    vector<int> constructArr(vector<int>& a) {
        vector<int> left(a.size(), 1);
        for(int i=1;i<a.size();i++){
            left[i] = a[i-1]*left[i-1];
        }
        vector<int> right(a.size(), 1);
        for(int i=a.size()-2;i>=0;i--){
            right[i] = a[i+1]*right[i+1];
        }
        vector<int> res(a.size(), 1);
        for(int i=0;i<a.size();i++){
            res[i] = left[i]*right[i];
        }
        return res;
    }
};

```

# 面试题 13. 机器人的运动范围

```c++
class Solution {
public:
    vector<int> constructArr(vector<int>& a) {
        vector<int> left(a.size(), 1);
        for(int i=1;i<a.size();i++){
            left[i] = a[i-1]*left[i-1];
        }
        vector<int> right(a.size(), 1);
        for(int i=a.size()-2;i>=0;i--){
            right[i] = a[i+1]*right[i+1];
        }
        vector<int> res(a.size(), 1);
        for(int i=0;i<a.size();i++){
            res[i] = left[i]*right[i];
        }
        return res;
    }
};
```

# 面试题 45. 把数组排成最小的数

对数字进行排序

```c++
string ab = a+b;
string ba = b+a;
return ab<ba;
```

```c++
class Solution {
public:
    static bool cmp(string a, string b){
        string ab = a+b;
        string ba = b+a;
        return ab<ba;
    }
    string minNumber(vector<int>& nums) {
        vector<string> str_num;
        for(int i=0;i<nums.size();i++){
            str_num.push_back(to_string(nums[i]));
        }
        sort(str_num.begin(), str_num.end(), cmp);
        string res = "";
        for(int i=0;i<str_num.size();i++){
            res = res + str_num[i];
        }
        return res;
    }
};
```

# !面试题 59 - II. 队列的最大值

一个 queue 还有一个 deque，queue push_back 的时候，将 max deque 中 back 中小于等于 value 的都 pop 走，在 pop_front 的时候，将 max deque 中 front 中等于 pop 出来的 value 的 pop 出去，总体维护的队列 back 到 front 的数据是有序增大的

```c++
class MaxQueue {
private:
    queue<int> q;
    deque<int> max_q;
public:
    MaxQueue() {

    }

    int max_value() {
        if(max_q.empty()){
            return -1;
        }
        return max_q.front();
    }
    void push_back(int value) {
        q.push(value);
        while(!max_q.empty() && max_q.back()<=value){
            max_q.pop_back();
        }
        max_q.push_back(value);
    }

    int pop_front() {
        if(q.empty()){
            return -1;
        }
        int ans = q.front();
        q.pop();
        while(!max_q.empty() && ans == max_q.front()){
            max_q.pop_front();
        }
        return ans;
    }
};
```

# 剑指 Offer 19. 正则表达式匹配

https://leetcode.cn/problems/zheng-ze-biao-da-shi-pi-pei-lcof
当`p[i-1] == '*'`的时候，总是可以删除一个前一个字符，即
` dp[i][j] |= dp[i][j-2];`，同时，如果如果当`s[i-1] == p[j-2] || p[j-2] == '.'`时，匹配
s 末尾的一个字符，将该字符扔掉，而该组合还可以继续进行匹配；

```c++
class Solution {
public:
    bool isMatch(string s, string p) {
        vector<vector<int>> dp(s.size()+1, vector<int>(p.size()+1, false));
        dp[0][0] = true;
        // 初始化首行 空串必须匹配 成对带* 的
        for(int i = 2; i <=p.size(); i += 2){
            dp[0][i] = (p[i-1] == '*' && dp[0][i-2]);
        }
        if(s.empty()) return dp[0][p.size()];

        for(int i=1;i<=s.size();i++){
            for(int j=1;j<=p.size();j++){
                if(s[i-1]==p[j-1] || p[j-1] == '.'){
                    dp[i][j] = dp[i-1][j-1];
                }else if(p[j - 1] == '*' && j>=2){
                    if(s[i-1] == p[j-2] || p[j-2] == '.'){
                        dp[i][j] = dp[i-1][j];
                    }
                    dp[i][j] |= dp[i][j-2];
                }

            }
        }
        return dp[s.size()][p.size()];
    }
};
```

# 剑指 Offer 51. 数组中的逆序对

https://leetcode.cn/problems/shu-zu-zhong-de-ni-xu-dui-lcof
利用 merge_sort

```c++
class Solution {
private:
    int ans = 0;
public:
    void merge(vector<int>& nums, int left, int mid, int right){
        vector<int> tp1(mid-left+1);
        vector<int> tp2(right-(mid+1)+1);
        for(int i=left;i<=mid;i++){
            tp1[i-left] = nums[i];
        }
        for(int i=mid+1;i<=right;i++){
            tp2[i-mid-1] = nums[i];
        }
        int i = left;
        int i1 = 0;
        int i2 = 0;
        while(i1<tp1.size() && i2<tp2.size()){
            if(tp1[i1] <= tp2[i2]){
                nums[i++] = tp1[i1++];
            }else{
                ans += (tp1.size()-i1);
                nums[i++] = tp2[i2++];
            }
        }
        while(i1<tp1.size()){
            nums[i++] = tp1[i1++];
        }
        while(i2<tp2.size()){
            nums[i++] = tp2[i2++];
        }
    }
    void merge_sort(vector<int>& nums, int left, int right){
        if(left>=right) return;
        int mid = left+(right-left)/2;
        merge_sort(nums, left, mid);
        merge_sort(nums, mid+1, right);
        merge(nums, left, mid, right);
    }

    int reversePairs(vector<int>& nums) {
        merge_sort(nums, 0, nums.size()-1);
        for(int i=0;i<nums.size();i++){
            printf("%d", nums[i]);
        }
        return ans;
    }
};
```

# 剑指 Offer 41. 数据流中的中位数

https://leetcode.cn/problems/shu-ju-liu-zhong-de-zhong-wei-shu-lcof/

```c++
class MedianFinder {
private:
    priority_queue<int, vector<int>, greater<int>> min_q;
    priority_queue<int, vector<int>, less<int>> max_q;
public:
    /** initialize your data structure here. */
    MedianFinder() {

    }

    void addNum(int num) {
        max_q.push(num);
        int tp = max_q.top();
        max_q.pop();
        min_q.push(tp);

        if(max_q.size() != min_q.size()){
            int tp = min_q.top();
            min_q.pop();
            max_q.push(tp);
        }
    }

    double findMedian() {
        if(min_q.size() == max_q.size()){
            return (1.0 * min_q.top() + max_q.top()) / 2;
        }
        return max_q.top();
    }
};
```

# 剑指 Offer 43. 1 ～ n 整数中 1 出现的次数

```c++
class Solution {
public:
    int countDigitOne(int n) {
        int left, now, right;
        long long k = 1;
        int ans = 0;
        while(left){
            now = (n % (k*10)) / k;
            right = n%k;
            left = n / (k*10);
            if(now == 0){
                // 12033; left:(0-11), right:(0-99);
                ans += left*k;
            }else if(now == 1){
                // 12133  left:(0-11), right(0-99), 当left=left时, right=(0, right)
                ans += left*k + right+ 1;
            }else{
                // 12933  left:(left)
                ans += (left+1)*k;
            }
            k = k*10;
        }
        return ans;
    }
};
```

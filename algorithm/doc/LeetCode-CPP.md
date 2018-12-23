# LeetCode

## 20

> 给定一个只包括 '('，')'，'{'，'}'，'['，']' 的字符串，判断字符串是否有效。

```cpp
#include <string>
#include <stack>

using namespace std;

class Solution {
public:

    bool isValid(string s) {
        stack<char> st;
        int i = 0;
        st.push('#');
        while (i < s.size()) {
            if ((s[i] == ')' && st.top() == '(') || (s[i] == ']' && st.top() == '[') ||
                (s[i] == '}' && st.top() == '{')) {
                i++;
                st.pop();
            } else {
                st.push(s[i]);
                i++;
            }
        }
        if (st.top() == '#')
            return true;
        return false;
    }

};
```

## 104

```cpp
struct TreeNode {
    int val;
    TreeNode *left;
    TreeNode *right;

    TreeNode(int x) : val(x), left(nullptr), right(nullptr) {} // NOLINT(google-explicit-constructor)

};

class Solution {
public:
    int max(int a, int b) {
        if (a > b)return a;
        else return b;
    }

    int maxDepth(TreeNode *root) {
        if (!root)
            return 0;
        return 1 + max(maxDepth(root->left), maxDepth(root->right));
    }
};
```

## 141

> 给定一个链表，判断链表中是否有环。

```cpp
#include <vector>

using namespace std;

class Solution {
private:
    struct ListNode {
        int val;
        ListNode *next;

        explicit ListNode(int x) : val(x), next(nullptr) {}
    };

public:
    bool hasCycle(ListNode *head) {
        ListNode *left = head, *right = head;
        while (right && right->next && right->next->next) {
            left = left->next;
            right = right->next->next;
            if (left == right) return true;
        }
        return false;
    }
};
```

## 144

> 给定一个二叉树，返回它的 前序 遍历。

```cpp
#include <vector>
#include <stack>

using namespace std;

class Solution {
private:
    struct TreeNode {
        int val;
        TreeNode *left;
        TreeNode *right;

        explicit TreeNode(int x) : val(x), left(nullptr), right(nullptr) {}
    };

public:
    vector<int> preorderTraversal(TreeNode *root) {
        vector<int> res;
        stack<TreeNode *> node_stack;
        if (root != nullptr) node_stack.push(root);
        while (!node_stack.empty()) {
            TreeNode *tmpNode = node_stack.top();
            node_stack.pop();
            res.push_back(tmpNode->val);
            if (tmpNode->right) node_stack.push(tmpNode->right);
            if (tmpNode->left) node_stack.push(tmpNode->left);
        }
        return res;
    }
};
```

## 148

> 在 $O(nlog^n)$ 时间复杂度和常数级空间复杂度下，对链表进行排序。

```cpp
#include <vector>
#include <algorithm>

using namespace std;

class Solution {
private:
    struct ListNode {
        int val;
        ListNode *next;

        explicit ListNode(int x) : val(x), next(nullptr) {}
    };

public:
    ListNode *sortList(ListNode *head) {
        ListNode *cur = head;
        vector<int> V;
        while (cur) {
            V.push_back(cur->val);
            cur = cur->next;
        }
        sort(V.begin(), V.end());
        cur = head;
        for (int i : V) {
            cur->val = i;
            cur = cur->next;
        }
        return head;
    }
};
```

## 155

> 设计一个支持 push，pop，top 操作，并能在常数时间内检索到最小元素的栈。

```cpp
#include <stack>

using namespace std;

class MinStack {
public:
    /** initialize your data structure here. */

    MinStack() {}

    void push(int x) {
        s1.push(x);
        if (s2.empty() || x <= s2.top())
            s2.push(x);
    }

    void pop() {
        if (s1.top() == s2.top())
            s2.pop();
        s1.pop();
    }

    int top() {
        return s1.top();
    }

    int getMin() {
        return s2.top();
    }

private:
    stack<int> s1, s2;
};
```

## 206

> 反转一个单链表。

```cpp
#include <iostream>

using namespace std;


class Solution {
private:
    struct ListNode {
        int val;
        ListNode *next;

        explicit ListNode(int x) : val(x), next(nullptr) {}
    };

public:
    ListNode *reverseList(ListNode *head) {
        if (head == nullptr) {
            cout << "The list is empty." << endl;
            return nullptr;
        }
        if (head->next == nullptr) {
            return head;
        }
        ListNode *newHead = reverseList(head->next);
        head->next->next = head;
        head->next = nullptr;
        return newHead;
    }
};
```

## 292

```cpp
class Solution {
public:
    bool canWinNim(int n) {
        return (n % 4 != 0);
    }
};
```

## 344

```cpp
#include <string>

using namespace std;

class Solution {
public:

    string reverseString(string s) { // NOLINT(performance-unnecessary-value-param)
        string ret;
        if (s.empty()) return ret;
        string::const_iterator tail = s.cend();
        do {
            tail--;
            ret.push_back(*tail);
        } while (tail != s.cbegin());
        return ret;
    }

};
```

## 622

> 设计循环队列实现

```cpp
class MyCircularQueue {
private:
    int *data;
    int head;
    int tail;
    int len;
    int count;

public:
    /** Initialize your data structure here. Set the size of the queue to be k. */
    explicit MyCircularQueue(int k) {
        data = new int[k];
        head = 0;
        tail = 0;
        len = k;
        count = 0;
    }

    /** Insert an element into the circular queue. Return true if the operation is successful. */
    bool enQueue(int value) {
        if (isFull()) return false;
        else {
            data[tail] = value;
            count++;
            tail = (tail + 1) % len;
            return true;
        }
    }

    /** Delete an element from the circular queue. Return true if the operation is successful. */
    bool deQueue() {
        if (isEmpty()) return false;
        else {
            head = (head + 1) % len;
            count--;
            return true;
        }
    }

    /** Get the front item from the queue. */
    int Front() {
        if (isEmpty()) return -1;
        else return data[head];
    }

    /** Get the last item from the queue. */
    int Rear() {
        if (isEmpty()) return -1;
        else {
            int tmp = tail == 0 ? (len - 1) : (tail - 1);
            return data[tmp];
        }
    }

    /** Checks whether the circular queue is empty or not. */
    bool isEmpty() {
        return count == 0;
    }

    /** Checks whether the circular queue is full or not. */
    bool isFull() {
        return count == len;
    }
};
```

## 704

> 给定一个 n 个元素有序的（升序）整型数组 nums 和一个目标值 target
> 写一个函数搜索 nums 中的 target，如果目标值存在返回下标，否则返回 -1。

```cpp
#include <vector>

using namespace std;

class Solution {
public:
    int search(vector<int> &nums, int target) {
        int left = 0;
        auto right = static_cast<int>(nums.size() - 1);
        while (left <= right) {
            int mid = (right + left) / 2;
            if (nums[mid] == target)
                return mid;
            else if (nums[mid] > target)
                right = mid - 1;
            else left = mid + 1;
        }
        return -1;
    }
};
```

## 707

> 设计链表的实现

```cpp
#include <cstdlib>

class MyLinkedList {
private:
    struct ListNode {
        int val;
        ListNode *next;

        explicit ListNode(int x) : val(x), next(nullptr) {}
    };

public:
    /** Initialize your data structure here. */
    MyLinkedList() {
        LinkedList = nullptr;
    }

    ListNode *LinkedList;

    /** Get the value of the index-th node in the linked list. If the index is invalid, return -1. */
    int get(int index) {
        int i = 0;
        ListNode *head = LinkedList;
        while (head && i < index) {
            head = head->next;
            i++;
        }
        if (head && i == index) return head->val;
        else return -1;
    }

    /** Add a node of value val before the first element of the linked list. After the insertion, the new node will be the first node of the linked list. */
    void addAtHead(int val) {
        auto *head = (ListNode *) malloc(sizeof(ListNode));
        head->next = LinkedList;
        head->val = val;
        LinkedList = head;
    }

    /** Append a node of value val to the last element of the linked list. */
    void addAtTail(int val) {
        ListNode *head = LinkedList;
        auto *tmp = (ListNode *) malloc(sizeof(ListNode));
        tmp->next = nullptr;
        tmp->val = val;
        if (!head) {
            LinkedList = tmp;
            return;
        }
        while (head->next)
            head = head->next;
        head->next = tmp;
    }

    /** Add a node of value val before the index-th node in the linked list. If index equals to the length of linked list, the node will be appended to the end of linked list. If index is greater than the length, the node will not be inserted. */
    void addAtIndex(int index, int val) {
        int i = 0;
        ListNode *head = LinkedList;
        if (!head && index == 0) {
            auto *tmp = (ListNode *) malloc(sizeof(ListNode));
            tmp->val = val;
            tmp->next = nullptr;
            LinkedList = tmp;
            return;
        }
        while (head && i < index - 1) {
            head = head->next;
            i++;
        }
        if (head && head->next == nullptr) {
            auto *tmp = (ListNode *) malloc(sizeof(ListNode));
            tmp->val = val;
            tmp->next = nullptr;
            head->next = tmp;
        } else if (i == index - 1 && head && head->next) {
            auto *tmp = (ListNode *) malloc(sizeof(ListNode));
            tmp->val = val;
            tmp->next = head->next;
            head->next = tmp;
        }
    }

    /** Delete the index-th node in the linked list, if the index is valid. */
    void deleteAtIndex(int index) {
        ListNode *head = LinkedList;
        int i = 0;
        while (head && i < index - 1) {
            head = head->next;
            i++;
        }
        if (head == nullptr) return;
        if (head->next == nullptr && index == 0) {
            LinkedList = nullptr;
            return;
        }
        if (head->next) {
            ListNode *tmp = head->next;
            head->next = tmp->next;
            free(tmp);
        }
    }
};
```

## 709

> 实现函数 `ToLowerCase()`，该函数接收一个字符串参数 str，并将该字符串中的大写字母转换成小写字母，之后返回新的字符串。

```cpp
class Solution {
public:
    string toLowerCase(string str) {
        for (char &i : str)
            if ((i >= 'A') && (i <= 'Z'))
                i = i + ('a' - 'A');
        return str;
    }
};
```

## 728

> 自除数 是指可以被它包含的每一位数除尽的数。给定上边界和下边界数字，输出一个列表，列表的元素是边界（含边界）内所有的自除数。

```cpp
#include <vector>
#include <string>
#include <sstream>

using namespace std;

class Solution {
public:
    vector<int> selfDividingNumbers(int left, int right) {
        vector<int> v;
        for (int i = left; i <= right; ++i) {
            stringstream ss;
            ss << i;
            string s = ss.str();
            int l = static_cast<int>(s.length()), t = 0;
            for (int j = 0; j < l; ++j) {
                if (s[j] == '0')
                    break;
                if (i % (s[j] - '0') == 0)
                    t++;
            }
            if (l == t)
                v.push_back(i);
        }
        return v;
    }
};
```

# 1
> Multidimensional arrays can be stored in row major order, as in C++, or in column major order, as in Fortran. Develop the access functions for both of these arrangements for ­three-​­dimensional arrays.

```c
// array.c
#include <stddef.h>

void *access_row(void *arr, size_t size, int dim1_i, int dim2_i, int dim3_i, int dim2_l, int dim3_l) {
    size_t offset_ele = dim1_i * dim2_l * dim3_l + dim2_i * dim3_l + dim3_i;

    size_t offset_bytes = offset_ele * size;
    char *byte_arr = (char *)arr;

    return (void *)(byte_arr + offset_bytes);
}

void *access_col(void *arr, size_t size, int dim1_i, int dim2_i, int dim3_i, int dim1_l, int dim2_l) {
    size_t offset_ele = dim3_i * dim2_l * dim1_l + dim2_i * dim1_l + dim1_i;

    size_t offset_bytes = offset_ele * size;
    char *byte_arr = (char *)arr;

    return (void *)(byte_arr + offset_bytes);
}

```

# 2.
> Suppose someone designed a stack abstract data type in which the function `top` returned an access path (or pointer) rather than returning a copy of the top element. This is not a true data abstraction. Why? Give an example that illustrates the problem.

回傳指標相當於破壞封裝，洩漏實作給使用者，讓使用者有能力破壞抽象資料型別的不變性。

```cpp
// stack.cpp
#include <iostream>
#include <stack> 

template<typename T>
class BadStack {
private:
    T arr[10];
    int top_index = -1;
public:
    void push(T val) { arr[++top_index] = val; }
    void pop() { --top_index; }
    T* top() { return &arr[top_index]; } 
};

int main() {
    BadStack<int> s;
    s.push(20);

    std::cout << "Top element is: " << *(s.top()) << std::endl;  // top = 20

    int* internal_ptr = s.top(); // top = 20
    *internal_ptr = 999; // top = 999ee

    std::cout << "Top element is now: " << *(s.top()) << std::endl;  // top = 999 !!!
}

```

# 3.
> Compare the multiple inheritance of C++ with that provided by interfaces in Java.

C++ 繼承所有東西，包括實作（implementation）和狀態（state），且允許多重繼承。這導致當兩個父類別都有同樣的狀態，子類別內部就會有兩份來自共同祖先的狀態，導致名稱歧義。

Java 預設只支援單一類別繼承，透過界面（interface）機制達到部份多重繼承的效果。界面只包含方法宣告以及命名常數。避免 C++ 多重繼承的缺點，但無法程式碼重用。
# 4.
> What is the primary reason why all Java objects have a common ancestor?

為了統一性以及代碼重用。所有的類別都是根類別的子類別，這樣就可以直接在根類別裡定義常用的方法，例如 `ToString`, `Finalize`。
# 5.
> What are the differences between a C++ abstract class and a Java interface?

C++ 抽象類別可以包含實例變數以及已定義的方法。而 Java 的界面只包含命名常數和方法宣告，只定義一個類別的規格，不提供程式碼重用。
# 6.
> Study and explain the issue of why Java does not include C++’s destructor even though both these languages have constructors.

C++ 因為不包含垃圾回收，所以需要解構函式明確處理資源釋放，而 Java 所有物件都是明確的堆積動態（explicit heap dynamic），並且內建垃圾回收機制，程式設計師不需要明確處理資源釋放，也因此 Java 不需要解構函式。
[2008B](https://codeforces.com/problemset/problem/2008/B)	[Square or Not](https://codeforces.com/problemset/problem/2008/B)

思路：

1、先判断是否为平方数；
2、遍历正方形，计算`(i,j)`所对应在 str 中的下标，判断正方形边缘是否为1，内部是否为0。

```c++
#include <iostream>
#include<cmath>

using  namespace std;

bool isPerfectSquare(long long num) {
    if (num < 0) return false;  // 负数不是平方数
    long long sqrtNum = sqrt(num);  // 计算整数平方根
    return sqrtNum * sqrtNum == num;  // 检查平方是否等于原数
}

int main() {
    int n;
    cin >> n;
    for (int i = 1; i <= n; i++) {
        int x;
        string str;
        cin >> x;
        cin >> str;
        if (isPerfectSquare(x)) {
            int sqrtNum = sqrt(x);
            bool flag = true;
            for (int j = 0; j < sqrtNum; ++j) {
                for (int k = 0; k < sqrtNum; ++k) {
                    int index = j * sqrtNum + k;
                    if (j == 0 || k == 0 || j == sqrtNum - 1 || k == sqrtNum - 1) {
                        if (str[index] != '1') {
                            flag = false;
                        } 
                    } else {
                        if (str[index] != '0') {
                            flag = false;
                        }
                    }
                }
            }
            if (flag) {
                cout << "Yes" << endl; 
            } else {
                cout << "No" << endl;
            }
        } else {
            cout << "No" << endl;
        }
    }
    return 0;
}
```

<!-- ##{"script":"<script src='https://blog.meekdai.com/Gmeek/plugins/GmeekVercount.js'></script>"}## -->
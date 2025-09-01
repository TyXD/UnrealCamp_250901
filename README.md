# [내일배움캠프] 해시, 동적 계획법

## 학습 키워드

>## 해쉬와 문자열

>## 동적 계획법

## 학습 내용

## 해시와 문자열 처리

### 해시

해시 테이블은 키를 사용하여 값을 빠르게 접글할 수 있도록 데이터를 구성하는 자료구조입니다.

예를 들어 문자열 배열을 통해 문자열과 연결된 다른 데이터를 찾을려면, 

문자열을 하나씩 비교해야합니다.

이때 해시 태이블을 이용하면 문자열을 인덱스로 변환하는 해시 함수를 통해 

테이블을 순차 탐색, 비교 하지 않아도 쉽게 데이터를 찾을 수 있게 해줍니다.

### 해시 함수

해시 함수는 키를 해시 테이블의 인덱스로 변환하는 역할을 합니다.

해시 함수를 구현할 때 고려해야하는 주요 요소들은 다음과 같습니다.

>### 1. 해시 함수가 반환하는 인덱스는 해시 테이블의 크기보다 작아야합니다.

인덱스가 테이블의 범위를 벗어나면 접근할 수 없기 때문입니다.

>### 2. 충돌이 최대한 적게 발생하도록 함수를 설계해야합니다.

서로 다른 키를 해시 함수에 적용할때, 

동일한 인덱스가 반환되지 않도록 설계해야합니다.

충돌을 완전히 피할 수는 없지만, 

충돌의 빈도가 낮으면서 해시 값일 균일하게 분포되도록 함수를 구현해야합니다.

## 자주 사용하는 해시 함수

### 나눗셈법

키를 소수로 나눈 나머지를 활용합니다.

		- h(x) = x mod k; (x =  키, k = 소수)

k를 소수로 쓰는 이유는 소수가 아닌 수를 사용하면,

특정 패턴의 키에서 충돌이 주기적으로 발생합니다.

예를 들어 k를 12이고 x가 3의 배수로 주어지면

x를 나눌때 주기적으로 해시값이 충돌합니다.

```cpp

#include <iostream>
using namespace std;

int hashFunction(int key, int prime) {
    return key % prime;
}

int main() {
    int key = 1234;
    int prime = 11; // 소수 사용
    int hashValue = hashFunction(key, prime);

    cout << "해시값: " << hashValue << endl;

    return 0;
}

/*
출력결과
해시값: 2
*/
```

### 곱셉법

나눗셈법과 다르게 곱셈법은 소수를 사용하지 않고 

실수(무리수)를 활용하여 해시 인덱스를 계산하는 방법입니다.

		- h(x) = (((x*a)mod 1)*m)
		- m은 최대 버킷의 개수
		- a는 황금비 같은 무리수

이 방식은 소수를 구할 필요가 없고, 

키값이 균등하게 분포될 가능성이 높아 실용적으로 자주 사용됩니다.

곱셈법의 과정은 다음과 같습니다.

>### 1. 키에 황금비를 곱합니다.

황금비는 1.61803.....입니다.

이를 키에 곱하면 정수 부분과 소수 부분이 나옵니다.

>### 2. 1에서 구한 값에 모듈러 1을 취합니다.

모듈러 1은 해당값의 정수 부분을 없애고 소수 부문만 값으로 사용합니다.

>### 3. 2에서 구한 값에 m을 곱해서 정수 부분을 구합니다.

2에서 구한 값은 소수이므로 이는 1보다 작은 값입니다.

m을 곱하고 정수를 취하면 0 ~ (m-1)의 값 중 하나가 나옵니다.

이를 해시 테이블의 인덱스로 활용합니다.

``` cpp

#include <iostream>
#include <cmath>

using namespace std;

int hashMultiplication(int key, int m)
{
    double A = (sqrt(5.0) - 1) / 2; // 황금비의 역수 ≈ 0.6180339887
    double frac = fmod(key * A, 1.0); // 소수 부분만 추출
    return int(m*frac);
}
int main()
{
    int key = 12345;
    int m = 16;

    int hashValue = hashMultiplication(key, m);
    cout << "해시 인덱스: " << hashValue << endl;
    return 0;
}

/*
출력결과
해시 인덱스: 10
*/
```

### 문자열 해싱(다항식 해싱)

해시 테이블의 키가 문자열인 경우 사용하는 기법입니다.

문자열을 정수로 변환하여 해시 값을 계산하는 방식이며,

일반적으로 다항식 해싱(polynomial rolling hash) 방식이 사용됩니다.

		- hash(s) = (s[0] + s[1]*p + s[2]*p^2 + ... + s[n-1]*p^(n-1))mod m

		- p는 31입니다..(메르센 소수)
		- s[n]은 문자열

이 방식은 문자열 내 문자의 위치와 값을 고려하여 계산하므로,

서로 다른 문자열이 동일한 해시 값을 가질 확률이 매우 낮습니다.

다만 해당식을 그대로 사용하면 중간에 오버플로우가 방생할 확률이 높습니다.

그렇기에 모듈러 연산을 중간에 이용하는 공식을 사용합니다

		- (a+b)%c = (a%c + b%c)%c

최종적으로 코드에서 사용하는 다항식 해싱은 다음과 같습니다.

		- hash(s) = (s[0] % m + s[1]*p % m + s[2]*p^2 % m + ... + s[n-1]*p^(n-1) % m) % m

```cpp

#include <iostream>
#include <string>

using namespace std;

int simple_hash(string s)
{
    const int m = 400;
    int hash = 0;
    int base = 31;
    for (int i = 0; i < s.length(); i++)
    {
        int value = s[i] - 'a' + 1;
        hash += value * base;
        base *= 31;
    }

    return hash % m;
}
int main()
{
    string str1 = "aa";
    string str2 = "b";

    // 해시값 출력
    cout << "문자열 '" << str1 << "'의 해시값: " << simple_hash(str1) << endl;
    cout << "문자열 '" << str2 << "'의 해시값: " << simple_hash(str2) << endl;

    return 0;
}
/*
	문자열 str1의 해시값 : 192
	문자열 str2의 해시값 = 62
*/
```

## 해시 충돌

### 체이닝

체이닝은 해시 충돌이 발생할 때, 해당 해시 버킷에 값을

연결 리스트(Linked List) 형태로 저장하는 방식입니다.

```cpp

// 목적: 해시 테이블에서 체이닝(Chaining)을 이용하여 값을 저장하고 출력하는 예제
// 동작: 동일한 해시 인덱스를 가진 값들은 연결 리스트로 연결됨

#include <iostream>
#include <list>
using namespace std;

const int TABLE_SIZE = 5;

class HashTable {
    list<int> table[TABLE_SIZE];

public:
    void insert(int key) {
        int index = key % TABLE_SIZE;
        table[index].push_back(key);
    }

    void display() {
        for (int i = 0; i < TABLE_SIZE; ++i) {
            cout << i << ": ";
            for (int val : table[i]) {
                cout << val << " -> ";
            }
            cout << "NULL" << endl;
        }
    }
};

int main() {
    HashTable ht;
    ht.insert(1);
    ht.insert(6);
    ht.insert(11);
    ht.insert(2);
    ht.insert(7);

    ht.display();

    return 0;
}

/*
출력결과
0: NULL
1: 1 -> 6 -> 11 -> NULL
2: 2 -> 7 -> NULL
3: NULL
4: NULL
*/
```

### 개방 주소법

개방 주소법은 해시 테이블 내 빈 버킷을 찾아

충돌 값을 삽입하는 방식입니다.

뱔도의 연결리스트나 외부 메모리를 사용하지 않고

추가적인 메모리 할당이 없습니다.

다만 충돌이 많아지면 클러스터링이 발생합니다.

		- h(k, i) = (h(k)+i) mod m
		- m은 최대 버킷의 수
		- i는 탐사 횟수


```cpp

#include <iostream>
#include <vector> // 동적 배열 사용
#include <stdexcept> // 예외 처리 (선택 사항)

using namespace std;

const int TABLE_SIZE = 5;
const int EMPTY_SLOT = -1; // 빈 슬롯 표시 값

class HashTable {
    vector<int> table; // 해시 테이블 저장소 (벡터 사용)
    int current_size; // 현재 저장된 요소 개수

public:
    // 생성자: 테이블 초기화
    HashTable() : table(TABLE_SIZE, EMPTY_SLOT), current_size(0) {}

    // 선형 탐사 삽입
    void insert(int key) {
        // 테이블이 가득 찼는지 확인
        if (current_size >= TABLE_SIZE) {
             cerr << "오류: 해시 테이블 가득 참 (" << key << " 삽입 불가)" << endl;
             return;
        }

        int index = key % TABLE_SIZE; // 초기 해시 인덱스 계산
        int original_index = index;

        // 빈 슬롯 찾기 (선형 탐사)
        while (table[index] != EMPTY_SLOT) {
            index = (index + 1) % TABLE_SIZE; // 다음 인덱스로 이동 (테이블 끝->처음)
            if (index == original_index) { // 한 바퀴 돌았는지 확인 (이론상 불필요)
                 cerr << "오류: 삽입 중 무한 루프 가능성. 키: " << key << endl;
                 return;
            }
        }

        // 빈 슬롯에 키 삽입 및 크기 증가
        table[index] = key;
        current_size++;
    }

    // 해시 테이블 내용 출력
    void display() {
        cout << "--- 해시 테이블 (선형 탐사) ---" << endl;
        for (int i = 0; i < TABLE_SIZE; ++i) {
            cout << i << ": ";
            if (table[i] != EMPTY_SLOT) {
                cout << table[i]; // 값이 있으면 출력
            } else {
                cout << "EMPTY"; // 비어있으면 EMPTY 출력
            }
            cout << endl;
        }
        cout << "-----------------------------" << endl;
    }

    // 인덱스 1의 상태 출력
    void display_index_one() {
        cout << "--- 인덱스 1의 상태 ---" << endl;
        cout << "1: ";
        if (table[1] != EMPTY_SLOT) {
            cout << table[1];
        } else {
            cout << "EMPTY";
        }
        cout << endl;
        cout << "-----------------------" << endl;
    }
};

int main() {
    HashTable ht;
    ht.insert(1);
    ht.insert(6);
    ht.insert(11);
    ht.insert(2);
    ht.insert(7);

    ht.display(); // 전체 테이블 출력
    ht.display_index_one(); // 인덱스 1의 상태 출력

    return 0;
}

/*
최종 테이블 상태 (삽입 완료 후): [7, 1, 6, 11, 2]

예상 출력 결과:
--- 해시 테이블 (선형 탐사) ---
0: 7
1: 1
2: 6
3: 11
4: 2
-----------------------------
--- 인덱스 1의 상태 ---
1: 1
-----------------------
*/
```

### 이중 해싱

충돌이 발생 했을때를 대비해 해시 함수를 2개 사용합니다.

두 번째 해시 함수의 역할은 첫 번째 해시 함수가 충돌이 발생하면

해당 위치를 정할때 사용합니다.

이중 해싱은 선형 탐사나 이차 탐사에 비해 클러스터링 현상이 적고,

충톨 처리 효율 높아 성능이 우수한 방식으로 평가합니다.

		- h(k, i) = (h1(k) + i * h2(k)) mod m
		- m은 최대 버킷
		- i는 탐사 횟수

## 동적 계획법

복잡한 문제를 더 작고 관리하기 쉬운 하위 문제로

분할하여 해결하는 방법론입니다.

동적 계획법을 효과적으로 적용하기 위한 두 가지 전제 조건이 있습니다.

## 최적 부분 구조

전체 문제의 최적 해결척이 하위 문제들의

최적 해결책으로부터 구성될 수 있다는 것을 의미합니다.

### 중복 부분 문제

원래 문제의 해결 과정에서 동일한 하위 문제가

여러번 반복적으로 나타나는 경우를 이야기합니다.

### 동적 계획법을 풀기 위한 점화식 수립

동적 계획법으로 문제를 해결하는 과정은 다음과 같습니다.

>### 1. 최적 부분 구조 및 중복 부분 구조 문제인지 확인

>### 2. 기저 조건 확인

점화식이 적용되지 않는 가장 작은 문제의 해는 직접 계산하여,

이를 기저 조건으로 설정합니다.

>### 3. 점화식 수립

### 메모이제이션 기법

동적 계획법이 적용 가능한 문제에서 동적 계획법을 구현할 때,

메모이제이션 기법을 사용해서 동적 테이블을 구축합니다.

메모이제이션은 한 번 계산한 결과를 저장하고,

동일한 문제가 다시 등장했을 때 이를 재사용하는 기법입니다.

이때 저장되는 공간을 동적 테이블이라 하며, 주로 배열로 구현합니다.

## 최장 증가 부분 수열(LIS)

### 부분 수열

주어진 수열에서 일부 원소를 순서를 유지한 채 선택하여 만든 새로운 수열입니다.

원래 수열의 원소들을 중간에 생략할 수는 있지만, 원래 순서대로 나와야 합니다.

이 중 부분 수열 중에서 원소들이 오름차순으로 증가하고,

그 길이가 가장 긴 수열을 최장 증가 부분 수열이라 정의합니다.

```cpp

// 목적: 배열과 동적 계획법(DP)을 이용해 최장 증가 부분 수열(LIS)의 길이를 구한다 (O(N^2))
#include <iostream>
using namespace std;

int lis(int arr[], int n) {
    int dp[100] = {1}; // LIS 길이를 저장할 배열 (최대 100개까지)

    for (int i = 0; i < n; ++i)
        dp[i] = 1;

    for (int i = 1; i < n; ++i) {
        for (int j = 0; j < i; ++j) {
            if (arr[i] > arr[j] && dp[i] < dp[j] + 1)
                dp[i] = dp[j] + 1;
        }
    }

    int maxLen = 0;
    for (int i = 0; i < n; ++i)
        if (dp[i] > maxLen)
            maxLen = dp[i];

    return maxLen;
}

int main() {
    int arr[] = {1, 4, 2, 3, 5, 7};
    int n = sizeof(arr) / sizeof(arr[0]);
    cout << lis(arr, n) << endl;
    return 0;
}

// 출력결과
// 5
```
		- 반복문을 사용한 LIS 구현(메모이제이션 포함)

```cpp

// 목적: 배열과 재귀 + 메모이제이션을 이용해 최장 증가 부분 수열(LIS)의 길이를 구한다 (O(N^2))
#include <iostream>
#include <cstring> // memset 사용
using namespace std;

int dp[100];      // LIS 메모이제이션 배열
int arr[100];     // 입력 배열

int lis(int n) {
    if (dp[n] != -1) return dp[n];

    dp[n] = 1; // 최소 길이는 자기 자신 하나

    for (int i = 0; i < n; ++i) {
        if (arr[n] > arr[i]) {
            dp[n] = max(dp[n], lis(i) + 1);
        }
    }

    return dp[n];
}

int main() {
    int input[] = {1, 4, 2, 3, 5, 7};
    int n = sizeof(input) / sizeof(input[0]);

    memcpy(arr, input, sizeof(input));
    memset(dp, -1, sizeof(dp));

    int maxLen = 0;
    for (int i = 0; i < n; ++i) {
        maxLen = max(maxLen, lis(i));
    }

    cout << maxLen << endl;
    return 0;
}

// 출력결과
// 5
```

		- 재귀 함수를 사용한 LIS 구현(메모이제이션 포함)







#해시함수 #동적 계획법 #내일배움캠프 #언리얼5기

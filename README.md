## Laboratory work V

Данная лабораторная работа посвещена изучению фреймворков для тестирования на примере **GTest**

```sh
$ open https://github.com/google/googletest
```

[![Coverage Status](https://coveralls.io/repos/github/Nejelia/lab05/badge.svg)](https://coveralls.io/github/Nejelia/lab05)

## Homework

Копируем репрозиторий tp-labs №5
```console
$ git clone https://github.com/tp-labs/lab05 lab05
Клонирование в «lab05»...
remote: Enumerating objects: 137, done.
remote: Counting objects: 100% (25/25), done.
remote: Compressing objects: 100% (9/9), done.
remote: Total 137 (delta 18), reused 16 (delta 16), pack-reused 112
Получение объектов: 100% (137/137), 918.92 КиБ | 1.18 МиБ/с, готово.
Определение изменений: 100% (60/60), готово.
$ cd lab05
$ git remote remove origin
$ git remote add origin https://github.com/Nejelia/lab05
```

Добавляем *googletest*
```console
$ git submodule add https://github.com/google/googletest

```

Добавляем *tests/test1.cpp*
```console
$ mkdir tests
$ cd tests
$ cat > test1.cpp
#include "Account.h"
#include "Transaction.h"

#include <gtest/gtest.h>
#include <gmock/gmock.h>

class AccountMock : public Account {
public:
    AccountMock(int id, int balance) : Account(id, balance) {}
    MOCK_CONST_METHOD0(GetBalance, int());
    MOCK_METHOD1(ChangeBalance, void(int diff));
    MOCK_METHOD0(Lock, void());
    MOCK_METHOD0(Unlock, void());
};
class TransactionMock : public Transaction {
public:
    MOCK_METHOD3(Make, bool(Account& from, Account& to, int sum));
};

TEST(Account, Mock) {
    AccountMock acc(1, 100);
    EXPECT_CALL(acc, GetBalance()).Times(1);
    EXPECT_CALL(acc, ChangeBalance(testing::_)).Times(2);
    EXPECT_CALL(acc, Lock()).Times(2);
    EXPECT_CALL(acc, Unlock()).Times(1);
    acc.GetBalance();
    acc.ChangeBalance(100); // throw
    acc.Lock();
    acc.ChangeBalance(100);
    acc.Lock(); // throw
    acc.Unlock();
}

TEST(Account, SimpleTest) {
    Account acc(1, 100);
    EXPECT_EQ(acc.id(), 1);
    EXPECT_EQ(acc.GetBalance(), 100);
    EXPECT_THROW(acc.ChangeBalance(100), std::runtime_error);
    EXPECT_NO_THROW(acc.Lock());
    acc.ChangeBalance(100);
    EXPECT_EQ(acc.GetBalance(), 200);
    EXPECT_THROW(acc.Lock(), std::runtime_error);
}

TEST(Transaction, Mock) {
    TransactionMock tr;
    Account ac1(1, 50);
    Account ac2(2, 500);
    EXPECT_CALL(tr, Make(testing::_, testing::_, testing::_))
    .Times(6);
    tr.set_fee(100);
    tr.Make(ac1, ac2, 199);
    tr.Make(ac2, ac1, 500);
    tr.Make(ac2, ac1, 300);
    tr.Make(ac1, ac1, 0); 
    tr.Make(ac1, ac2, -1); 
    tr.Make(ac1, ac2, 99); 
}

TEST(Transaction, SimpleTest) {
    Transaction tr;
    Account ac1(1, 50);
    Account ac2(2, 500);
    tr.set_fee(100);
    EXPECT_EQ(tr.fee(), 100);
    EXPECT_THROW(tr.Make(ac1, ac1, 0), std::logic_error);
    EXPECT_THROW(tr.Make(ac1, ac2, -1), std::invalid_argument);
    EXPECT_THROW(tr.Make(ac1, ac2, 99), std::logic_error);
    EXPECT_FALSE(tr.Make(ac1, ac2, 199));
    EXPECT_FALSE(tr.Make(ac2, ac1, 500));
    EXPECT_FALSE(tr.Make(ac2, ac1, 300));
}
^Z
[1]+  Остановлен    cat > test1.cpp
```

Добавляем *test.yaml*
```console
$ mkdir .github
$ cd .github
$ mkdir workflows
$ cd workflows
$ cat > test.yaml
name: CMake


on:
 push:
  branches: [master]
 pull_request:
  branches: [master]

jobs:
 Tests:

  runs-on: ubuntu-latest

  steps:
  - uses: actions/checkout@v3

  - name: Adding googletest
    run: git clone https://github.com/google/googletest.git googletest -b release-1.11.0

  - name: Install lcov
    run: sudo apt-get install -y lcov

  - name: Config banking with tests
    run: cmake -H. -B ${{github.workspace}}/build -DBUILD_TESTS=ON

  - name: Build banking
    run: cmake --build ${{github.workspace}}/build

  - name: Run tests
    run: build/check
    
  - name: Do lcov stuff
    run: lcov -c -d build/CMakeFiles/banking.dir/banking/ --include *.cpp --output-file ./coverage/lcov.info    

  - name: Publish to coveralls.io
    uses: coverallsapp/github-action@v1.1.2
    with:
      github-token: ${{ secrets.GITHUB_TOKEN }}
      ^Z
[2]+  Остановлен    cat > test.yaml
```

Добавляем *coverage/Icov.info*
```console
$ mkdir coverage
$ cd coverage
$ cat > Icov.info
^Z
[3]+  Остановлен    cat > Icov.info
```

Пушим проект
```console
$ git add --all
$ git commit -m "First commit"
[master 93d3223] First commit
 5 files changed, 115 insertions(+)
 create mode 100644 .github/workflows/test.yaml
 create mode 100644 .gitmodules
 create mode 100644 coverage/Icov.info
 create mode 160000 googletest
 create mode 100644 tests/test1.cpp
$ git push origin master
Username for 'https://github.com': Nejelia
Password for 'https://Nejelia@github.com': 
Перечисление объектов: 147, готово.
Подсчет объектов: 100% (147/147), готово.
При сжатии изменений используется до 2 потоков
Сжатие объектов: 100% (79/79), готово.
Запись объектов: 100% (147/147), 920.56 КиБ | 13.54 МиБ/с, готово.
Всего 147 (изменений 61), повторно использовано 136 (изменений 60), повторно использовано пакетов 0
remote: Resolving deltas: 100% (61/61), done.
To https://github.com/Nejelia/lab05
 * [new branch]      master -> master
```

## Links

- [C++ CI: Travis, CMake, GTest, Coveralls & Appveyor](http://david-grs.github.io/cpp-clang-travis-cmake-gtest-coveralls-appveyor/)
- [Boost.Tests](http://www.boost.org/doc/libs/1_63_0/libs/test/doc/html/)
- [Catch](https://github.com/catchorg/Catch2)

```
Copyright (c) 2015-2021 The ISC Authors
```

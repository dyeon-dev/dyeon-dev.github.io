# 백트래킹 return과 exit(0)에 관한 고찰글 (java)

### 백트래킹 문제에서 재귀를 빠져나오는 부분에서 return을 써야할까? exit(0)을 써서 프로그램을 종료시켜야 할까?

스터디에서 백트래킹 문제를 푸는 날, 어떤 기준으로 조건문을 판단하여 함수가 종료되는지 답이 확실하게 나오지 않았습니다. 따라서 함께 풀었던 문제를 기준으로 return과 exit(0) 함수에 관한 고찰글을 작성하게 되었습니다.

일단 서칭을 해봤을 때, return과 exit(0)에 대한 차이점을 찾아보면 다음과 같이 나와있습니다.

#### return
- 해당 함수를 종료하는 키워드
- 종료 후 함수를 호출한 곳으로 이동
- main()함수에서 사용 시 프로그램 종료 (=exit()함수)

#### exit()
- 프로그램을 강제로 종료하는 함수
- 호출한 위치와 관계없이 프로그램 종료 


<hr>

### Code 1
> <https://www.acmicpc.net/problem/18428>

처음 해당 문제에 대한 코드를 짰을 때는 정답을 찾으면 바로 답을 출력하면서 exit(0) 함수로 프로그램을 종료시켰습니다.

재귀를 통해 답을 찾고나면 그 뒤의 구성들을 더이상 확인할 필요가 없기 때문에 System.exit(0);을 사용하여 불필요한 백트래킹과 계산을 방지합니다.

```java
// System.exit(0); 사용
private static void dfs(int cnt) {
    if (cnt == 3) {
        if (isSafe()) {
            System.out.println("YES");
            System.exit(0);
        }
        return;
    }
    for (int i = 0; i < n; i++) {
        for (int j = 0; j < n; j++) {
            if (arr[i][j] == 'X') {
                arr[i][j] = 'O'; 
                dfs(cnt + 1);
                arr[i][j] = 'X'; 
            }
        }
    }
}
```

만약 여기에서 System.exit(0);이 아닌 return;문을 넣는다면, 전체 프로그램이 아니라 현재 재귀호출만 종료하게 됩니다. 따라서 모든 구성에 대한 결과값이 출력됩니다. (오답)
```java
// return 사용
if (cnt == 3) {
  if (isSafe()) {
      System.out.println("YES");
      return;
  }
  return;
}
```
```
YES
YES
...
(25개의 결과값)
```


### 그럼 무조건 exit(0) 함수를 써야만 할까?
System.exit(0)은 JVM을 강제로 중단시키기 때문에 일부 자바 커뮤니티에서는 다른 옵션이 있는 한 종료 방법의 사용을 권장하지 않습니다.

따라서 위의 코드를 1. 플래그를 사용하거나 2. boolean 값을 반환하여 코드를 리팩토링 할 수 있습니다.
#### 1. 플래그 사용

```java
dfs(0);

if (!foundSolution) {
    System.out.println("NO");
}

private static void dfs(int cnt) {
  if (foundSolution) return;
  if (cnt == 3) {
      if (isSafe()) {
          System.out.println("YES");
          foundSolution = true;
      }
      return;
  }
}
```

#### 2. boolean 값으로 반환

```java
if (dfs(0)) {
      System.out.println("YES");
  } else {
      System.out.println("NO");
  }

  private static boolean dfs(int cnt) {
        if (cnt == 3) {
            return isSafe();
        }

        for (int i = 0; i < n; i++) {
            for (int j = 0; j < n; j++) {
                if (arr[i][j] == 'X') {
                    arr[i][j] = 'O';
                    if (dfs(cnt + 1)) {
                        return true;
                    }
                    arr[i][j] = 'X';
                }
            }
        }
        return false;
  }
```

이러한 방식들을 사용하면 exit(0)과 마찬가지로 추가 탐색을 중단하는데 유용하게 사용될 수 있습니다.

[전체코드보기](https://github.com/UREKA-Algorithm-Study/KimDaYeon/blob/main/week5/bj_18428.java)

### Code2
><https://www.acmicpc.net/problem/2580>

또 다른 문제에서도 재귀 부분에서 exit()가 아닌 return은 왜 안되는지 살펴보겠습니다.

```java
private static void sudoku(int row, int col) {
		// 0~8행 모두 채워졌을 경우 스도쿠 출력
		if (row == 9) {
			for (int[] a : sudoku) {
				for (int num : a) {
					System.out.print(num+" ");
				}
				System.out.println();
			}
			System.exit(0); // return;쓰면 틀림.. 왜징 (시간초과 x 틀렸습니다 뜸)
		}

		// 0~8열 탐색하여 한 행이 다 채워졌을 경우 다음 행의 0번째 열부터 시작
		if (col == 9) {
			sudoku(row+1,0);
			return;
		}
		
		if (sudoku[row][col] == 0) {
			// 1~9 중 가능한 수 탐색 (가로줄,세로줄, 3*3 정사각형 고려)
			for (int i=1; i<=9; i++) {
				boolean garo = Garo(row, col, i);
				boolean sero = Sero(row, col, i);
				boolean square = Square(row ,col, i);
				
				// 가로줄, 세로줄, 정사각형에서 모두 중복되지 않을 경우
				if (garo && sero && square) {
					sudoku[row][col] = i;
					sudoku(row,col+1);
				}
			}
			sudoku[row][col]=0; // 백트래킹
			return;
		}
		sudoku(row, col+1);
	}
```
이 코드도 마찬가지로, return을 사용하면 현재 재귀 호출만 종료되고 재귀 호출 전체 시퀀스는 종료되지 않습니다. 

중첩된 재귀로 문제를 풀었기 때문에 정답을 찾으면 System.exit(0);으로 모든 재귀를 종료해야 합니다.

즉, 스도쿠 구성을 찾았다면 추가적인 재귀 호출을 계속할 필요가 없습니다.
return(현재 재귀만 종료되고 그 뒤의 재귀가 호출됨)을 사용하면 정답이 출력될지언정, 다른 가능성을 계속 탐색하며 잘못된 동작이나 출력을 발생시킬 수 있습니다. 

(해당 문제의 테스트 케이스는 return을 사용해도 정답이 출력된다. 왜냐하면 재귀 호출에서 return에 접근했을 때 이전의 값들은 모두 호출을 통해 검색이 종료되기 때문. 하지만 백준에 제출하면 오답 처리)

### return으로 사용하기
위에서 봤던 두번째 방법인 boolean 값으로 반환하는 방법을 사용해도 좋습니다.

스도쿠를 출력하는 부분을 함수로 빼서 관리하는 것입니다.
```java
        // Sudoku 시작
        solveSudoku(0, 0);

        // 완성된 Sudoku 출력
        printSudoku();

// Sudoku 시작
private static boolean solveSudoku(int row, int col) {
  // 0~8행 모두 채워졌을 경우 스도쿠 출력
    if (row == 9) {
        return true;
    }
    {
    ...(other codes)

          // 가로줄, 세로줄, 정사각형에서 모두 중복되지 않을 경우

          if (garo && sero && square) {
            sudoku[row][col] = i;
            
            // 재귀 호출
            if (solveSudoku(row, col + 1)) {
                return true;
            }
            
            sudoku[row][col] = 0; // 백트래킹
        }
      return false;
    }
  return solveSudoku(row, col + 1);
}

// 완성된 Sudoku 출력
private static void printSudoku() {
    for (int[] row : sudoku) {
        for (int num : row) {
            System.out.print(num + " ");
        }
        System.out.println();
    }
}
```

이처럼 exit 대신 return을 재귀 흐름을 제어하는 데 사용하면 좀 더 구조화된 접근 방식으로 보장됩니다.

[전체코드보기](https://github.com/UREKA-Algorithm-Study/KimDaYeon/blob/main/week5/bj_2580.java)

## 결론
return으로 사용하는 것이 코드의 유연성과 유지 관리에 좋지만, 재귀를 통한 백트래킹 방식에서는 exit()가 상당히 요긴하게 쓰입니다 🙂 !
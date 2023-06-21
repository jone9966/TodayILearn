# 벨만 방정식

벨만 방정식은 현재 상태의 가치함수를 다음 상태의 가치함수를 통해 나타낸 식

![상태 가치함수](./image/rl-dp1.png)

![행동 가치함수](./image/rl-dp2.png)

위 식에 기대값의 개념이 들어가기 때문에 벨만 기대 방정식이라고 한다.

벨만 기대 방정식을 통해 가치함수를 계속 업데이트하면 현재의 정책을 따라갔을 때 얻을 실제 보상의 참 기댓값을 구할 수 있다.

벨만 기대 방정식을 풀면서 구한 가치함수를 통해 정책을 업데이트하다 보면 최적의 정책을 찾을 수 있다.

최적의 정책은 최적의 가치함수를 받게하는 정책이며 그때 가치함수 사이의 관계식을 벨만 최적 방정식이라고 한다.

![벨만 기대 방정식](./image/rl-dp3.png)

![벨만 최적 방적식](./image/rl-dp4.png)



# 다이나믹 프로그래밍

다이나믹 프로그래밍은 하나의 큰 문제를 여러 개의 작은 문제로 나누어 푸는 것

### Policy Iteration

벨만 기대 방정식을 사용해 정책을 평가(Evaluate)하고 이를 바탕으로 정책을 발전(Improve)시킨다.

![Policy Iteration](./image/rl-dp5.png)

### 예시
앞서 구현한 그리드월드 환경에서 policy evaluation을 수행하는 코드이다.
현재 가치함수를 바탕으로 벨만 기대방정식을 통해 가치 함수를 갱신한다.
계속해서 갱신하다 보면 갱신이 더이상 안될때가 있는데 그 때의 가치함수가 현재 정책에서의 참 가치함수이다.
```
def policy_evaluation(v):
    v_plus = np.zeros_like(v, dtype=float)
    
    for i in range(5):
        for j in range(5):
            if (i, j) == (2, 2): continue
            p_ij = p[i][j]
            for a in range(len(A)):
                i_, j_ = np.add([i, j], A[a])
                if i_ < 0: i_ = 0
                if j_ < 0: j_ = 0
                if i_ > 4: i_ = 4
                if j_ > 4: j_ = 4

                v_plus[i][j] += p_ij[a] * (rwd[i_][j_] + df * v[i_][j_])
    return v_plus
    
v = [[0. for _ in range(5)] for _ in range(5)]
k = 0
while True:
    k += 1
    v_plus = policy_evaluation(v)
    if np.array_equal(v, v_plus):
        print(v.round(2))
        break
    else:
        v = v_plus
```

정책 평가를 통해 얻은 가치 함수를 바탕으로 greedy 하게 행동하도록 정책을 개선하면 최적 정책을 찾을 수 있다.
```
def policy_improvement(p):
    for i in range(5):
        for j in range(5):
            if (i, j) == (2, 2): continue
            p_ij = p[i][j]
            v_next = []
            for a in range(len(A)):
                i_, j_ = np.add([i, j], A[a])
                if i_ < 0: i_ = 0
                if j_ < 0: j_ = 0
                if i_ > 4: i_ = 4
                if j_ > 4: j_ = 4
                v_next.append(rwd[i_][j_] + df * v[i_][j_])

            max_v = np.where(v_next == np.max(v_next))[0]
            per = 1 / len(max_v)

            p_next = [0.] * 4
            for k in max_v:
                p_next[k] = per
            p[i][j] = p_next
    return p
```



### Value Iteration

Policy Iteration 과 같이 명시적인 정책을 발전시키는 것이 아니라 가치함수 안에 내재된 정책이 있다고 가정하고 가치함수만 업데이트 하여 최적의 정책을 찾는 것

![Value Iteration](./image/rl-dp6.png)

### 예시
value iteration은 최적 정책을 찾기 위해 벨만 최적 방정식을 반복 수행하여 각 상태의 가치 함수를 갱신한다.
```
def value_iteration(v):
    v_plus = np.zeros_like(v, dtype=float)
    
    for i in range(5):
        for j in range(5):
            if (i, j) == (2, 2): continue
            p_ij = p[i][j]
            max_v = -1
            for a in range(len(A)):
                i_, j_ = np.add([i, j], A[a])
                if i_ < 0: i_ = 0
                if j_ < 0: j_ = 0
                if i_ > 4: i_ = 4
                if j_ > 4: j_ = 4

                value = rwd[i_][j_] + df * v[i_][j_]
                max_v = value if value > max_v else max_v
            v_plus[i][j] = max_v
    return v_plus
```


다이나믹 프로그래밍의 한계

- 다이나믹 프로그래밍은 학습이 아닌 계산이다.
- 문제의 상태와 행동이 늘어날수록 계산량이 급격히 증가한다.
- 다이나믹 프로그래밍은 환경(보상, 상태 변환 확률)을 알고 있다는 가정이 필요하기 때문에 그 외의 문제에서는 사용할 수 없다.
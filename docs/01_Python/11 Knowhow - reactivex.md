# ReactiveX

## 1개의 rx에서 혹은 1개의 task에서 operator 별 thread 적용하는 방법

저는 보통 서버에 접속해서 데이터를 가져오는 일을 많이 하는데, 서버마다 순차적으로 순회하면 오래 걸리니 병렬로 진행하고 싶었습니다. 그러나 rx task를 많이 만들고 그 task를 각각 따로 실행하는 형태가 tutorial로 제공되었습니다. 저는 IP 리스트가 있고 각각 서버에 접속해서 데이터를 긁어오고 싶었으므로 아래와 같이 작성하였습니다.

```python
import reactivex as rx
import reactivex.operators as ops
from reactivex.scheduler import ThreadPoolScheduler
from multiprocessing import cpu_count
import random
import time

random.seed(0)

def delay(x):

    sec = random.randint(10, 30) * 0.1
    time.sleep(sec)
    return f"{x} : {sec:.2f}"

pool = ThreadPoolScheduler(cpu_count())

rx.from_iterable([1,2,3,4,5,6]).pipe(
    # just 부분에서 pool을 ops.subscribe_on로 변경했을 때 미묘하게 빠른 기분이었습니다.
    ops.flat_map(lambda x: rx.just(x, pool).pipe(ps.map(lambda x: delay(x)))),
    ops.to_list()
).subscribe(
    on_next = lambda x: print(x),
    on_error = lambda x: print(x)
)

pool.executor.shutdown()

```
# Command Pattern

명령을 commander에게 전송하여 작업을 실행하는 메서드를 분리합니다.

---

Command Pattern으로 특정 작업을 실행하는 객체들을 메소드를 호출하는 객체와 분리할 수 있습니다.

온라인 식품 배달 플랫폼이 있고, 사용자는 주문을 하거나, 추적하고 취소할 수 있습니다.

```js
class OrderManager() {
  constructor() {
    this.orders = []
  }

  placeOrder(order, id) {
    this.orders.push(id)
    return `You have successfully ordered ${order} (${id})`;
  }

  trackOrder(id) {
    return `Your order ${id} will arrive in 20 minutes.`
  }

  cancelOrder(id) {
    this.orders = this.orders.filter(order => order.id !== id)
    return `You have canceled your order ${id}`
  }
}
```

`OrderManager` 클래스에는 `placeOrder`, `trackOrder`, `cancelOrder` 메소드가 있습니다. 이러한 메소드를 직접 사용하는 것이 완전히 유효한 자바스크립트입니다!

```js
const manager = new OrderManager();

manager.placeOrder("Pad Thai", "1234");
manager.trackOrder("1234");
manager.cancelOrder("1234");
```

그러나, 특정 메소드의 이름을 나중에 변경하거나 기능을 변경할 때 `manager` 인스턴스에 직접 메소드를 호출하는 것은 단점이 있습니다. 

`placeOrder`로 메시지를 전송하는 대신, `addOrder`로 이름을 수정했습니다! 이제 `placeOrder`를 더이상 어떤곳에서도 호출하면 안됩니다. 그러나 이것은 큰 규모의 시스템에서는 번거로운 일일수도 있습니다.

대신에 `manager` 객체에서 메소드를 분리하고 각 명령마다 명령 함수를 생성하고 싶습니다.

`OrderManager` 클래스를 리팽터링해봅시다! `placeOrder`, `cancelOrder`, `trackOrder` 메소드를 내부에 포함하지말고, `execute`라는 단 하나의 메소드만을 둡니다.

`execute`는 받은 명령을 실행할 것입니다.

각 명령은 `manager` 내부의 `orders`에 접근해야 하므로 `execute` 메소드의 첫번째 인자로 `orders`를 전달합니다.

```js
class OrderManager {
  constructor() {
    this.orders = [];
  }

  execute(command, ...args) {
    return command.execute(this.orders, ...args);
  }
}
```

We need to create three Commands for the order manager:

order manager에 사용할 세 개의 명령들을 생성합니다.

- `PlaceOrderCommand`
- `CancelOrderCommand`
- `TrackOrderCommand`

```js
class Command {
  constructor(execute) {
    this.execute = execute;
  }
}

function PlaceOrderCommand(order, id) {
  return new Command(orders => {
    orders.push(id);
    return `You have successfully ordered ${order} (${id})`;
  });
}

function CancelOrderCommand(id) {
  return new Command(orders => {
    orders = orders.filter(order => order.id !== id);
    return `You have canceled your order ${id}`;
  });
}

function TrackOrderCommand(id) {
  return new Command(() => `Your order ${id} will arrive in 20 minutes.`);
}
```

좋습니다! `OrderManager` 인스턴스에 직접 결합된 메소드를 사용하는 것에서 이제 `OrderManager의` `execute` 메소드를 통해 호출되는 함수로 분리되었습니다.

```js
class OrderManager {
  constructor() {
    this.orders = [];
  }

  execute(command, ...args) {
    return command.execute(this.orders, ...args);
  }
}

class Command {
  constructor(execute) {
    this.execute = execute;
  }
}

function PlaceOrderCommand(order, id) {
  return new Command(orders => {
    orders.push(id);
    console.log(`You have successfully ordered ${order} (${id})`);
  });
}

function CancelOrderCommand(id) {
  return new Command(orders => {
    orders = orders.filter(order => order.id !== id);
    console.log(`You have canceled your order ${id}`);
  });
}

function TrackOrderCommand(id) {
  return new Command(() =>
    console.log(`Your order ${id} will arrive in 20 minutes.`)
  );
}

const manager = new OrderManager();

manager.execute(new PlaceOrderCommand("Pad Thai", "1234"));
manager.execute(new TrackOrderCommand("1234"));
manager.execute(new CancelOrderCommand("1234"));
```

## Pros

Command pattern을 사용하면 기능을 실행하는 객체에서 메소드를 분리할 수 있습니다. 특정 수명이 있거나 대기하다가 특정 시간에 실행되는 명령들을 잘 다룰 수 있습니다. 

## Cons
command pattern 사용 사례는 다소 제한적이고 불필요한 boilerplate를 자주 생성합니다.

---

## 첨언

커맨드 패턴은 **실행할 기능**을 추상화한다. 

> 이벤트가 발생했을 때 실행될 기능이 다양하면서도 변경이 필요한 경우에 이벤트를 발생시키는 클래스를 변경하지 않고 재사용하고자 할 때 유용하다.  
> 실행될 기능을 캡슐화함으로써 기능의 실행을 요구하는 호출자(Invoker) 클래스와 실제 기능을 실행하는 수신자(Receiver) 클래스 사이의 의존성을 제거한다.  
> [출처](https://gmlwjd9405.github.io/2018/07/07/command-pattern.html)


### History do/undo 기능 구현

```ts
class HistoryItem {
    constructor(private readonly text: string, private readonly date: Date) {}

    private get dateTime() {
        return `${this.date.toLocaleDateString()} ${this.date.toLocaleTimeString()}`
    }

    get content() {
        return `[${this.dateTime}] ${this.text}`
    }
}

interface Command {
    execute: (histories: HistoryItem[]) => void
}

class HistoryManager {
    histories: HistoryItem[] = []

    execute(cmd: Command) {
        cmd.execute(this.histories)
    }
}

const BEHAVIOR_LIST = {
    CUT: 'cut',
    FOLD: 'fold',
    PASTE: 'paste'
} as const

type BahaviorList = typeof BEHAVIOR_LIST[keyof typeof BEHAVIOR_LIST]

class Do implements Command {
    constructor(private behavior: BahaviorList) {}
    execute(histories: HistoryItem[]) {
        const history = new HistoryItem(this.behavior, new Date());
        console.log(history.content);
        histories.push(history);
  }
}

class UnDo implements Command {
    execute(histories: HistoryItem[]) {
    const prevBehavior = histories.pop();
    if (!prevBehavior) {
        console.error("더이상 취소할 작업이 없습니다.");
        return;
    }
    console.log(`undo ${prevBehavior.content}`);
  }
}

const manager = new HistoryManager()

manager.execute(new Do(BEHAVIOR_LIST.CUT));
manager.execute(new Do(BEHAVIOR_LIST.FOLD));
manager.execute(new UnDo());
manager.execute(new UnDo());
manager.execute(new UnDo());
```
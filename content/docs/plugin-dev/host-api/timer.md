---
title: Timer 接口
---

用于在宿主侧注册 timeout / interval 定时器，并在触发时以插件事件回调回来。

## 接口定义

```wit
interface timer {
    set-timeout: func(delay-ms: u64, payload: string) -> future<u64>;
    set-interval: func(interval-ms: u64, payload: string) -> future<u64>;
    clear-timer: func(timer-id: u64) -> future;
}
```

## 函数

### set-timeout

- 参数：
  - `delay-ms: u64` 延迟毫秒数。
  - `payload: string` 定时器自定义载荷。
- 返回：`future<u64>`，成功时返回 `timer-id`。

### set-interval

- 参数：
  - `interval-ms: u64` 间隔毫秒数。
  - `payload: string` 定时器自定义载荷。
- 返回：`future<u64>`，成功时返回 `timer-id`。

### clear-timer

- 参数：`timer-id: u64`
- 返回：`future<()>`。

## 注意事项

- 宿主会保证 `delay-ms` / `interval-ms` 最小为 `1ms`，避免出现 0 间隔。
- `set-timeout` 触发一次后会自动从宿主侧移除。
- `set-interval` 会持续触发，直到你调用 `clear-timer`，或插件实例被销毁。
- `clear-timer` 即使传入不存在的 `timer-id`，当前也不会报错，只是静默返回。

## 事件回调

定时器触发后，插件会在 `on_event` 中收到 `event::EventType::Timer`，`event-payload` 当前是一个 JSON 字符串，形如：

```json
{
  "timerId": 1,
  "kind": "timeout",
  "payload": "refresh-devices"
}
```

- `timerId`：宿主分配的定时器 ID。
- `kind`：`timeout` 或 `interval`。
- `payload`：你在 `set-timeout` / `set-interval` 里原样传入的字符串。

## 示例

```rust tab="Rust"
use crate::astrobox::psys_host::timer;

pub async fn arm_refresh_timeout() -> u64 {
    timer::set_timeout(3_000, "refresh-devices").await
}

pub async fn stop_timer(timer_id: u64) {
    timer::clear_timer(timer_id).await;
}
```

```go tab="Go"
package plugin

import timer "astroboxplugin/bindings/astrobox_psys_host_timer"

func armRefreshTimeout() uint64 {
	return timer.SetTimeout(3000, "refresh-devices").Read()
}

func stopTimer(timerID uint64) {
	timer.ClearTimer(timerID).Read()
}
```

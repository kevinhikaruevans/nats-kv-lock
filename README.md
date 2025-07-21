# nats-kv-lock
A simple, distributed lock using NATS.io

## Notes

This lock follows [the exclusive locking example](https://docs.nats.io/nats-concepts/jetstream/key-value-store/kv_walkthrough#create-aka-exclusive-locking) in the NATS JetStream docs. 
This lock does NOT guarantee order, i.e. it is an unfair lock.

## Installation

Assuming you already have NATS installed, you can simply `pip install nats-kv-lock`

## Example

All you need to do is create a KV bucket, then initialize an instance of `NatsKvLock` using that bucket with a shared `lock_name`.

```python
import asyncio

from nats import connect
from nats_kv_lock import NatsKvLock

async def main():
    print('connecting...')
    nc = await connect()
    print('connected!')
    
    js = nc.jetstream()
    try:
        kv = await js.key_value(bucket='my_locks')
    except:
        kv = await js.create_key_value(bucket='my_locks')
    
    my_lock = NatsKvLock(kv, 'my_lock')

    print('acquiring lock...')
    
    async with my_lock:
        print('acquired! doing work...')
        await asyncio.sleep(10)
        print('releasing lock...')

    print('released lock!')

if __name__ == '__main__':
    asyncio.run(main())
```

### With a timeout

If the lock needs a locking timeout, you can pass in a timeout into `acquire()`,

```python
my_lock = NatsKvLock(kv, 'my_lock')

was_locked = await x.acquire(5.0)

if was_locked:
   await asyncio.sleep(1.0)

await x.release()
```

Using a `try-finally` block would be a good choice here too.

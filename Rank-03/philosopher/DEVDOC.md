# Developer Documentation

## 1. Overview

This project solves Dining Philosophers in two ways:

- Mandatory (`philo`): shared-memory model with threads and mutexes.
- Bonus (`philo_bonus`): multi-process model with semaphores.

The core goals are:

1. Correct synchronization (no data races / deadlocks in normal operation).
2. Accurate timing for death, eating, sleeping.
3. Clean shutdown when one philosopher dies or when meal limits are satisfied.

---

## 2. Mandatory (`philo`) internals

### 2.1 Startup sequence

1. `main.c`
   - validates and loads CLI arguments via `init_data`.
   - allocates and initializes philosopher data via `init_philos`.
   - initializes `interrupt_mutex`.
   - starts simulation through `make_threads`.
2. Cleanup destroys mutexes and frees allocated memory.

### 2.2 Main structures

- `t_data`
  - global timing parameters and flags (`someone_died`, `finished`, `all_ate`, `interrupt_flag`).
  - shared synchronization primitives (`print`, `dead_mutex`, `meals_mutex`, etc.).
  - fork mutex array (`forks`).
- `t_philo`
  - per-philosopher state (`id`, `last_meal`, `meals_eaten`, fork indexes).
  - per-philosopher `meal_mutex` for safe meal-time/state updates.

### 2.3 Threading model

- One thread per philosopher runs the life routine.
- One monitor thread checks:
  - death conditions (`time_to_die` exceeded)
  - optional meal completion condition (`n_meals` reached for all)

### 2.4 Synchronization strategy

- One mutex per fork controls fork ownership.
- Dedicated mutexes protect shared flags and logging.
- Message printing is serialized through `print` mutex.
- Fork acquisition order is split by philosopher parity (even/odd logic), reducing circular-wait risk.

### 2.5 Lifecycle loop

Each philosopher repeatedly performs:

1. take forks
2. eat (update `last_meal`, increment meals)
3. release forks
4. sleep
5. think

Loop exits when monitor signals termination conditions.

---

## 3. Bonus (`philo_bonus`) internals

### 3.1 Startup sequence

1. `main.c`
   - validates arguments and initializes semaphores/data (`init_data`).
   - forks one child process per philosopher (`create_philosophers`).
2. Parent waits for meal completion (if configured) and/or death signal.
3. Parent terminates all children through `kill_all`, then runs `cleanup`.

### 3.2 Process model

- Parent process: orchestration and termination.
- Child process (one per philosopher): executes philosopher routine.
- Inside each child, a monitor thread watches starvation (`monitor_routine`).

### 3.3 Semaphore design

Global named semaphores include:

- `forks`: counting semaphore for fork resources.
- `print`: serializes output.
- `death`: termination signal propagation.
- `full`: meal completion signaling.
- `start`: synchronized start barrier.
- `eating_limit`: optional control around meal completion flow.

Per philosopher:

- `meal_lock`: protects `last_meal` and meal counters in child context.

### 3.4 Child lifecycle

Typical child flow:

1. wait for synchronized start
2. take forks (`sem_wait`)
3. print and eat
4. sleep
5. think
6. repeat until death/termination or meal target reached

---

## 4. Error handling and edge cases

- Invalid CLI arguments return early with usage/error output.
- Single philosopher scenarios are handled explicitly.
- On any hard failure during initialization, the program exits safely.
- Parent/monitor logic ensures global termination after death detection.

---

## 5. Build and run

### Mandatory

```bash
cd philo
make
./philo 5 800 200 200
```

### Bonus

```bash
cd philo_bonus
make
./philo_bonus 5 800 200 200 7
```

---

## 6. File-level map (quick)

### Mandatory (`philo`)

- `main.c`: bootstrap + cleanup
- `init_data.c`, `init_philos.c`: configuration and philosopher setup
- `make_threads.c`: thread creation/join
- `routine.c`, `take_forks.c`: philosopher actions
- `monitor.c`, `check_death_loop.c`: simulation monitors
- `philo_utils.c`, `help_functions.c`: utility and printing/time helpers

### Bonus (`philo_bonus`)

- `main.c`: parent orchestration
- `init_data.c`: semaphore and config init
- `create_philosophers.c`: `fork()` flow
- `philo_routine.c`: philosopher child behavior
- `monitor_routine.c`: starvation monitor thread
- `kill_all.c`, `cleanup.c`: shutdown path
- `sem_utils.c`, `utils.c`: semaphore name/time/print helpers

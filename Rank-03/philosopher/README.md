# Philosophers (42)

This repository contains two implementations of the Dining Philosophers problem:

- `philo/` (mandatory): threads + mutexes
- `philo_bonus/` (bonus): processes + POSIX semaphores

Both versions simulate philosophers that alternate between eating, sleeping, and thinking while preventing data races and handling death timing rules.

## Requirements

- Linux (or Unix-like OS)
- `cc` compiler
- `make`
- `pthread` support
- POSIX semaphore support (for bonus)

## Build

### Mandatory

```bash
cd philo
make
```

Produces executable: `./philo`

### Bonus

```bash
cd philo_bonus
make
```

Produces executable: `./philo_bonus`

## Usage

### Mandatory

```bash
./philo number_of_philosophers time_to_die time_to_eat time_to_sleep [number_of_times_each_philosopher_must_eat]
```

### Bonus

```bash
./philo_bonus number_of_philosophers time_to_die time_to_eat time_to_sleep [number_of_times_each_philosopher_must_eat]
```

All time values are in milliseconds.

## Example

```bash
./philo 5 800 200 200
./philo_bonus 5 800 200 200 7
```

## Output format

Runtime logs follow the standard format:

```text
<timestamp_in_ms> <philo_id> <action>
```

Typical actions:

- `has taken a fork`
- `is eating`
- `is sleeping`
- `is thinking`
- `died`

## Project layout

- `philo/`: mandatory implementation
- `philo_bonus/`: bonus implementation
- `philo_tester/`: tester utilities and generated result logs

## Documentation

- Developer documentation: `DEVDOC.md`
- Mandatory flow: `philo/philo_flow.txt`
- Bonus flow: `philo_bonus/philo_bonus_flow.txt`

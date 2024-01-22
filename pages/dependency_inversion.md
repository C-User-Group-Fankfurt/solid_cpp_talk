---
layout: center
---

# Time for a review...

---
layout: two-cols
---


<template v-slot:default>

## PROs

- interfaces for `Actor` and `DrivingModeAwareActor` are now separated
- implementations for both are decoupled

## CONs

- `DrivingModeAwareActor` require internal logic to select own `limits`
- poor testability of this logic
- smells like Single-Responsibility is not ensured anymore

</template>
<template v-slot:right>

```cpp {all|11-12}
int main(int, char **) {

  ...

  DrivingSystem 
   driving_system(sensor, planner,
                  {power_train, brake, 
                   steering_wheel});

  const auto driving_mode = DrivingMode::normal;
  brake->set_driving_mode(driving_mode);
  steering_wheel->set_driving_mode(driving_mode);

  driving_system.one_cycle();
}
```

</template>

---

# Extract Logic for Actor Limits

- introduction of `ActorLimitHandler` (implemented as singleton):
```cpp {1,7,10,16|1-5,17-18|all}
class ActorLimitHandler {
public:
    static ActorLimitHandler& get_instance () {
        static ActorLimitHandler actor_limiter;
        return actor_limiter;
    }
    void set_driving_mode(const DrivingMode &driving_mode) {
        current_driving_mode = driving_mode;
    }
    Acceleration get_current_deceleration_limit() {
        if (current_driving_mode == DrivingMode::normal)
            return deceleration_limit;
        else
            return unlimited_deceleration{std::numeric_limits<double>::lowest()};
    }
    Torque get_current_torque_limit(){...};
private:
    ActorLimitHandler() = default;
    Acceleration deceleration_limit{Acceleration{MetresPerSquareSecond{21.0}}};
    Torque torque_limit{Torque{NewtonMetre{3.0}}};
    DrivingMode current_driving_mode{DrivingMode::normal};
};
```

---

# Usage of Actor Limit Handler
- `DrivingModeAwareActor` gets obsolete
```cpp {1-9|6|11-22}
class Brake final : public Actor {
 public:
  Brake() = default;

  void control_vehicle(const Trajectory &) override {
    auto current_deceleration_limit = ActorLimitHandler::get_instance().get_current_deceleration_limit();
    std::cout << current_deceleration_limit << std::endl;
  };
};
...
int main(int, char **) {
  ...
  auto driving_mode = DrivingMode::normal;
  ActorLimitHandler::get_instance().set_driving_mode(driving_mode);

  driving_system.one_cycle();

  driving_mode = DrivingMode::emergency;
  ActorLimitHandler::get_instance().set_driving_mode(driving_mode);

  driving_system.one_cycle();
}
```

---

# The SOLID Principles

- **S**: Single-Responsibility Principle
- **O**: Open-Closed Principle
- **L**: Liskov Substitution Principle
- **I**: Interface Segregation Principle
- **D**: **Dependency Inversion Principle**

---

# SOLI**D**: Dependency Inversion Principle

- > The most flexible systems are those in which source code dependencies refer only to abstractions, not to concretions[^3]
- low-level implementations shall depend on high-level abstractions, not the other way around!
- architectural detail: abstractions shall be owned by the high-level, not the low-level



[^3]: : Martin, Robert C. (2017). Clean Architecture: A Craftsman's Guide to Software Structure and Design. Pearson.


---

# Dependencies of the Actors
- Actors have to depend on the implementation of `ActorLimitHandler`
- if `ActorLimitHandler` changes, `Brake` has to be re-compiled
- clear violation of the Dependency Inversion Principle

Possible Solution: 
- Actors define (and own) an interface to receive their current limit
- a low-level module (like the `ActorLimitHandler`) has to implement & fulfil the interface for each actor


---

# Dependency Inversion for Actors
```cpp {1|2,4-5,13|1-14|16-24}
using DecelerationLimitCallback = std::function<Acceleration()>;
class Brake final : public Actor {
 public:
  explicit Brake(DecelerationLimitCallback get_deceleration_limit) : get_current_deceleration_limit(
          std::move(get_deceleration_limit)) {};

  void control_vehicle(const Trajectory &) override {
    auto current_deceleration_limit = get_current_deceleration_limit();
    std::cout << current_deceleration_limit << std::endl;
  };

 private:
    DecelerationLimitCallback get_current_deceleration_limit;
};

using TorqueLimitCallback = std::function<Torque()>;
class SteeringWheel final : public Actor {
 public:
  explicit SteeringWheel(TorqueLimitCallback get_torque_limit)
      : get_current_torque_limit(std::move(get_torque_limit)) {}
  void control_vehicle(const Trajectory &) override {/*...*/};
 private:
    TorqueLimitCallback get_current_torque_limit;
};
```

---

# Driving System
```cpp {5-6|7-8|9-10|15-23}
int main(int, char **) {
  ...
  auto power_train = std::make_shared<PowerTrain>(
      Acceleration{MetresPerSquareSecond(13.6)});
  auto actor_limiter = std::make_shared<ActorLimitHandler>(Acceleration{MetresPerSquareSecond{21.0}},
                                                           Torque{NewtonMetre{3.0}});
  auto brake = std::make_shared<Brake>(
          [&actor_limiter]() {return actor_limiter->get_current_deceleration_limit();});
  auto steering_wheel = std::make_shared<SteeringWheel>(
          [&actor_limiter]() {return actor_limiter->get_current_torque_limit();});

  DrivingSystem driving_system(sensor, planner,
                               {power_train, brake, steering_wheel});

  auto driving_mode = DrivingMode::normal;
  actor_limiter->set_driving_mode(driving_mode);

  driving_system.one_cycle();

  driving_mode = DrivingMode::emergency;
  actor_limiter->set_driving_mode(driving_mode);

  driving_system.one_cycle();
}
```

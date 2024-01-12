---
theme: default
background: https://source.unsplash.com/collection/94734566/1920x1080
class: text-center
highlighter: shikiji
lineNumbers: true
info: |
  ## SOLID Principles using Modern C++
drawings:
  persist: false
transition: slide-left
title: SOLID Principles using Modern C++
mdc: true
---

# SOLID Principles in Modern C++

---
transition: fade-out
---

# The SOLID Principles

- **S**: Single-Responsibility Principle
- **O**: Open-Closed Principle
- **L**: Liskov Substitution Principle
- **I**: Interface Segregation Principle
- **D**: Dependency Inversion Principle

[SOLID on Wikipedia](https://en.wikipedia.org/wiki/SOLID)

---

# S**O**LID: Open-Closed Principle

- > software entities (classes, modules, functions, etc.) should be open for extension, but closed for modification.[^1]
- It should not be possible or necessary to change the code of an entity
- Still it should be possible to change the behavior of the entity


[^1]: : Meyer, Bertrand (1988). Object-Oriented Software Construction. Prentice Hall.

---

# The Actor in a Real Vehicle

- The Power Train could be either a combustion engine or electrical motor
- The brake could support either just comfort brakes or emergency brakes
- Can we control the handbrake?
- The steering wheel could be controllable via steering wheel angles or steering moments
- Some systems without a steering system might not provide a steering wheel at all
- **It highly depends on the system required by the customer**

---

# The Actor in our Current Implementation

```cpp {all|3-5,10,14,16-18,22}
class Actors {
 public:
  void control_power_train(const Trajectory & /*trajectory*/) {}
  void control_brake(const Trajectory & /*trajectory*/) {}
  void control_steering_wheel(const Trajectory & /*trajectory*/) {}
};

class DrivingSystem {
public:  
  DrivingSystem(std::shared_ptr<Sensor> sensor,
                std::shared_ptr<Planner> planner,
                std::shared_ptr<Actors> actor) :
    ...
    actor->control_brake(vehicle_trajectory);
    actor->control_power_train(vehicle_trajectory);
    actor->control_steering_wheel(vehicle_trajectory);
  }
 private:
  ...
  std::shared_ptr<Actors> actor;
};
```
---

# The Open-Closed Principle for the Actor

- We could try to implement one `Actor` class which supports all use cases (Single Responsibility)
- We want to be able to use different `Actor` implementations without changing the code of the `DrivingSystem`
- Solution in object-oriented software: Provide different implementations via *abstractions*

---

# An Abstract Actor

```cpp {0-5|7-20}
class Actor {
 public:
  virtual ~Actor() = default;
  virtual void control_vehicle(const Trajectory &) = 0;
};

class PowerTrain : public Actor {
 public:
  void control_vehicle(const Trajectory &) override {};
};

class Brake : public Actor {
 public:
  void control_vehicle(const Trajectory &) override {};
};

class SteeringWheel : public Actor {
 public:
  void control_vehicle(const Trajectory &) override {};
};
```

---

# Using the Abstract Actor in the Driving System

```cpp
class DrivingSystem {
 public:
  using Actors = std::vector<std::shared_ptr<Actor>>;

  DrivingSystem(std::shared_ptr<Sensor> sensor,
                std::shared_ptr<Planner> planner,
                Actors actors) :
  ...
  void one_cycle() {
    ...
    for (auto &actor : actors)
      actor->control_vehicle(vehicle_trajectory);
  }

 private:
  ...
  Actors actors;
};
```
---

# Bootstrapping the Actors

```cpp {all|5-7,10}
int main() {
  auto sensor = std::make_shared<Sensor>();
  auto planner = std::make_shared<Planner>();

  auto power_train = std::make_shared<PowerTrain>();
  auto brake = std::make_shared<Brake>();
  auto steering_wheel = std::make_shared<SteeringWheel>();

  DrivingSystem driving_system(sensor, planner,
                               {power_train, brake, steering_wheel});
  driving_system.one_cycle();
}
```

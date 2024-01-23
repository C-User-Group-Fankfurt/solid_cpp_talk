# The SOLID Principles

- **S**: Single-Responsibility Principle
- **O**: **Open-Closed Principle**
- **L**: Liskov Substitution Principle
- **I**: Interface Segregation Principle
- **D**: Dependency Inversion Principle

---

# Open-Closed Principle

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

# The Actors in our Current Implementation

```cpp {all|3-6,15,17,21}
class Actors {
 public:
  void control_vehicle(const Trajectory & trajectory) {
    control_power_train(trajectory);
    control_brake(trajectory);
    control_steering_wheel(trajectory);
  }
  ...
};

class DrivingSystem {
public:  
  DrivingSystem(std::shared_ptr<Sensor> sensor,
                std::shared_ptr<Planner> planner,
                std::shared_ptr<Actors> actors) :
    ...
    actors->control_vehicle(vehicle_trajectory);
  }
 private:
  ...
  std::shared_ptr<Actors> actors;
};
```
---

# The Open-Closed Principle for the Actors

- We could try to implement one `Actors` class which supports all use cases (Single Responsibility)
- We want to be able to use different `Actors` implementations without changing the code of the `DrivingSystem`
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


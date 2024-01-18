---

# The SOLID Principles

- **S**: Single-Responsibility Principle
- **O**: Open-Closed Principle
- **L**: **Liskov Substitution Principle**
- **I**: Interface Segregation Principle
- **D**: Dependency Inversion Principle

---

# Liskov Substitution Principle
<br>

![Liskov Substition](images/liskov_substitution.png) [^1] 

- If Class is a subclass of Base, then Class should behave like Base
- > Subtypes must be substitutable for their base types.
- With inheritance, we should model an **is-a** relation 

<br>
<br>

[^1]: : Liskov, Barbara; Wing, Jeannette (1994-11-01). A behavioral notion of subtyping. ACM Transactions on Programming Languages and Systems.

---

# Some derived Implications

- Preconditions cannot be strengthened in a subtype
- Postconditions cannot be weakened in a subtype
- Invariants of the super type must be preserved in a subtype

[^1]

## Image of invariant classes

[^1]: [Breaking Dependencies: The SOLID Principles - Klaus Iglberger - CppCon 2020](https://www.youtube.com/watch?v=Ntraj80qN2k)

---

# Moving to More Realistic Actors

- Actors can not generate arbitrary forces
- The range of forces is depending on the type of vehicle
- Open-Closed Principle: We want to change the limits from outside

## Image to explain actor limits: Steering Wheel

---

# The Naive Approach

```cpp {all|3,8,15,20}
class Actor {
...
virtual void set_limit(double /*limit*/) = 0;
};

class PowerTrain final : public Actor {
...
  void set_limit(double acceleration_limit_metres_per_second) override {
...
double max_acceleration_metres_per_second{0};
};

class Brake final : public Actor {
...
  void set_limit(double deceleration_limit_metres_per_second) override {
...

class SteeringWheel final : public Actor {
...
  void set_limit(double torque_limit_newton_metres) override {
...
```
---

# Following the Liskov Substitution Principle

Let's revert our mistake from before:
```cpp
class Actor {
 public:
  virtual ~Actor() = default;
  virtual void control_vehicle(const Trajectory & /*trajectory*/) = 0;
};
```

And move the limit to the constructor of the base class:
```cpp
class PowerTrain final : public Actor {
 public:
  explicit PowerTrain(const Acceleration& acceleration_limit) : acceleration_limit(acceleration_limit) {}
  void control_vehicle(const Trajectory &) override {};
 private :
  Acceleration acceleration_limit;
} 
```
---

# Following the Liskov Substitution Principle II

```cpp{all|3,11}
class Brake final : public Actor {
 public:
  explicit Brake(const Acceleration& deceleration_limit) : deceleration_limit(deceleration_limit) {};
  void control_vehicle(const Trajectory &) override {};
 private:
  Acceleration deceleration_limit;
};

class SteeringWheel final : public Actor {
 public:
  explicit SteeringWheel(const Torque& torque_limit) : torque_limit(torque_limit) {}
  void control_vehicle(const Trajectory &) override {};
 private:
  Torque torque_limit;
};
```
---

# Setting the Actual Limits

```cpp{all|4,6,8}
int main(int, char **) {
  ...
  auto power_train = std::make_shared<PowerTrain>(
    Acceleration{MetresPerSquareSecond{13.6}});
  auto brake = std::make_shared<Brake>(
    Acceleration{MetresPerSquareSecond{21.0}});
  auto steering_wheel = std::make_shared<SteeringWheel>(
    Torque{NewtonMetre{3.0}});

  DrivingSystem driving_system(sensor, planner,
                               {power_train, brake, steering_wheel});
  ...
}
```


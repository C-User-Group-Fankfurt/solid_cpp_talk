# The SOLID Principles

- **S**: Single-Responsibility Principle
- **O**: Open-Closed Principle
- **L**: **Liskov Substitution Principle**
- **I**: Interface Segregation Principle
- **D**: Dependency Inversion Principle

---

# Moving to More Realistic Actors

- Actors can not generate arbitrary forces
- The range of forces is depending on the type of vehicle
- Open-Closed Principle: We want to change the limits from outside

<img src="/actor_limits_single.png" alt="Steering Wheel Limit" class="m-10 h-60"/> 

---

# The Naive Approach

```cpp {all|1,4-5|5,10,17,22}
class Actor {
 public:
  virtual ~Actor() = default;
  virtual void control_vehicle(const Trajectory & /*trajectory*/) = 0;
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

# Liskov Substitution Principle
<br>

<img src="/liskov_substitution.png" alt="Liskov Substition"/> [^1]

<br>

- If Class is a subtype of Base, then Class should behave like Base
- > Subtypes must be substitutable for their base types.
- With inheritance, we should model an **is-a** relation 

<br>
<br>

[^1]: Liskov, Barbara; Wing, Jeannette (1994-11-01). A behavioral notion of subtyping. ACM Transactions on Programming Languages and Systems.

---

# Violation of Liskov Substitution Principle

<img src="/actor_limits.png" alt="Steering Wheel Limit" class="m-10 h-60"/> 

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
```cpp {all|3}
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

```cpp{all|3-5|7,8}
int main() {
  ...
  auto power_train = std::make_shared<PowerTrain>(Acceleration{MetresPerSquareSecond{13.6}});
  auto brake = std::make_shared<Brake>(Acceleration{MetresPerSquareSecond{21.0}});
  auto steering_wheel = std::make_shared<SteeringWheel>(Torque{NewtonMetre{3.0}});

  DrivingSystem driving_system(sensor, planner,
                               {power_train, brake, steering_wheel});
  ...
}
```


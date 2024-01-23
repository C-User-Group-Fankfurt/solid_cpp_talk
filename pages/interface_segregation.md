# Safety Aspects of a Real Driving System

- Normal driving (e.g. on a highway) requires only a limited functionality of our actors
- Any emergency situation (e.g. emergency braking into standstill) requires maximum actor capability
- As safety plays a major role we now want to handle these situations separately:

   a) normal situations - limitations for actors

   b) special situations - permission for actors to use extended limits


---

# Extension of the Abstract Actor


```cpp {0|1|3,6}
enum class DrivingMode { normal, emergency };

class Actor {
 public:
  virtual ~Actor() = default;
  virtual void control_vehicle(const Trajectory &, const DrivingMode &) = 0;
};
```
---

# Extension of Existing Brake Actor

```cpp {1|6-7|8,13-14|17-20}
class Brake final : public Actor {
 public:
  explicit Brake(const Acceleration &deceleration_limit) : deceleration_limit(
      deceleration_limit) {};

  void control_vehicle(const Trajectory &,
                       const DrivingMode &driving_mode) override {
    auto &current_limit = get_current_deceleration_limit(driving_mode);
    std::cout << current_limit << std::endl;
  };

 private:
  [[nodiscard]] const Acceleration &
  get_current_deceleration_limit(const DrivingMode &driving_mode) const {
    static const Acceleration
        unlimited_deceleration{std::numeric_limits<double>::lowest()};
    if (driving_mode == DrivingMode::normal)
      return deceleration_limit;
    else
      return unlimited_deceleration;
  }

  Acceleration deceleration_limit;
};
```

---

# Extension of Existing Powertrain Actor

```cpp {6,9-10|9-14|2-4,11-13}

class DrivingModeNotSupported : public std::logic_error {
  using std::logic_error::logic_error;
};

class PowerTrain final : public Actor {
 public:
  explicit PowerTrain(const Acceleration &acceleration_limit) : acceleration_limit(acceleration_limit) {}
  void control_vehicle(const Trajectory &,
                       const DrivingMode &driving_mode) override {
    if (driving_mode == DrivingMode::emergency)
      throw DrivingModeNotSupported(
          "Power train won't operate in emergency mode.");
  };
 private :
  Acceleration acceleration_limit;
};
```

---

# The SOLID Principles

- **S**: Single-Responsibility Principle
- **O**: Open-Closed Principle
- **L**: Liskov Substitution Principle
- **I**: **Interface Segregation Principle**
- **D**: Dependency Inversion Principle


---

# SOL**I**D: Interface Segregation Principle

- > clients should not be forced to depend on methods that they do not use.[^2]
- interfaces shall help to decouple, not introduce artifical coupling
- prefer multiple smaller interfaces to enable clients depending only on interfaces which are relevant for them
- interface segregation is an important special case of the Single-Responsibility Principle 



[^2]: : Martin, Robert C. (2013). Agile Software Development, Principles, Patterns, and Practices. Pearson.

---

# The Interface Segregation Principle for the Actor
- `Actor` implementations shall not be forced to know about the `DrivingMode`
- Interface changes shall only force `Actor` implementations to change for relevant reasons
- We could try to separate the `DrivingMode` related interface
- Only `Actor` implementations which use the `DrivingMode` really know about it
- Solution in object-oriented software: Provide multiple *abstractions*

---

# Separation of Abstract Actor

```cpp {1-5|7-10|1-5,12-18|7-10}
class Actor {
 public:
  virtual ~Actor() = default;
  virtual void control_vehicle(const Trajectory &) = 0;
};

class DrivingModeAwareActor : public Actor {
 public:
  virtual void set_driving_mode(const DrivingMode &) = 0;
};

class PowerTrain final : public Actor {
 public:
  explicit PowerTrain(const Acceleration &acceleration_limit) : acceleration_limit(acceleration_limit) {}
  void control_vehicle(const Trajectory &) override {...};
  ...
};
```

---

# Usage for Driving Mode aware Actors
```cpp {1-4|11-13,16,18,23|6-9}
class Brake final : public DrivingModeAwareActor {
 public:
  explicit Brake(const Acceleration &deceleration_limit) : deceleration_limit(
      deceleration_limit) {};

  void control_vehicle(const Trajectory &) override {
    auto &current_deceleration_limit = get_current_deceleration_limit();
    std::cout << current_deceleration_limit << std::endl;
  }

  void set_driving_mode(const DrivingMode &driving_mode) override {
    current_driving_mode = driving_mode;
  }

 private:
  [[nodiscard]] const Acceleration & get_current_deceleration_limit() const {
    ...
    if (current_driving_mode == DrivingMode::normal)
      ...
  }

  Acceleration deceleration_limit;
  DrivingMode current_driving_mode{DrivingMode::normal};
};
```
---

# Usage of the Driving Mode
```cpp {all|11-15}
int main(int, char **) {
  auto sensor = std::make_shared<Sensor>();
  auto planner = std::make_shared<Planner>();
  auto power_train = std::make_shared<PowerTrain>(Acceleration{MetresPerSquareSecond(13.6)});
  auto brake = std::make_shared<Brake>(Acceleration{MetresPerSquareSecond{21.0}});
  auto steering_wheel = std::make_shared<SteeringWheel>(Torque{NewtonMetre{3.0}});

  DrivingSystem driving_system(sensor, planner,
                               {power_train, brake, steering_wheel});

  const auto driving_mode = DrivingMode::normal;
  brake->set_driving_mode(driving_mode);
  steering_wheel->set_driving_mode(driving_mode);

  driving_system.one_cycle();
}
```

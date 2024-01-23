---

# The SOLID Principles

- **S**: **Single-Responsibility Principle**
- **O**: Open-Closed Principle
- **L**: Liskov Substitution Principle
- **I**: Interface Segregation Principle
- **D**: Dependency Inversion Principle

---

# Single Responsibility Principle

- > A class should have only one reason to change [^1]
- > Another wording for the Single Responsibility Principle is: Gather together the things that change for the same reasons. Separate those things that change for different reasons. [^2]

- Every class should have only one responsibility.
  - This avoids tight coupling.
  - Changing one responsibility shall not result in modifications of others.

<br>
<br>

[^1]: : Martin, Robert C. (2013). Agile Software Development, Principles, Patterns, and Practices. Pearson.
[^2]: : Martin, Robert C. (2014). The Single Responsibility Principle. The Clean Code Blog.

---

# Classic Violations
- Objects that can print / draw themselves
- Objects that can save / restore themselves

---

# Finally...
<br>
... some code

---

# Code Example - A Driving System

```cpp
struct EnvironmentModel {};
class Sensor {
 public:
  EnvironmentModel model_environment() {
    return EnvironmentModel{};
  }
};

class Planner {
 public:
  Trajectory plan_vehicle_behavior(const EnvironmentModel &/*environment*/) {
    return Trajectory{};
  }
};
```
---

# Code Example - A Driving System

```cpp {all|7,9,17}
namespace control_actuators {
void control_power_train(const ControlSignals &, PowerTrainConnection &) {}
void control_brake(const ControlSignals &, BrakeConnection &) {}
void control_steering_wheel(const ControlSignals &, SteeringConnection &) {}
}

class Trajectory {
 public:
  void control_vehicle() {
    using namespace control_actuators;
    control_power_train(internals, power_train_connection);
    control_brake(internals, brake_connection);
    control_steering_wheel(internals, steering_wheel_connection);
  };
 private:
  using TrajectoryData = int;
  TrajectoryData internals{};

  control_actuators::PowerTrainConnection power_train_connection{};
  control_actuators::BrakeConnection brake_connection{};
  control_actuators::SteeringConnection steering_wheel_connection{};
};
```
---

# Code Example - A Driving System

```cpp
class DrivingSystem {
 public:
  DrivingSystem(std::shared_ptr<Sensor> sensor,
                std::shared_ptr<Planner> planner) :
      sensor(std::move(sensor)),
      planner(std::move(planner)) {};

  void one_cycle() {
    auto environment_model = sensor->model_environment();
    auto vehicle_trajectory = planner->plan_vehicle_behavior(environment_model);
    vehicle_trajectory.control_vehicle();
  }

 private:
  std::shared_ptr<Sensor> sensor;
  std::shared_ptr<Planner> planner;
};
```
---

# Code Example - A Driving System

```cpp
int main() {
  auto sensor = std::make_shared<Sensor>();
  auto planner = std::make_shared<Planner>();

  DrivingSystem driving_system(
      sensor, planner);
  driving_system.one_cycle();
}
```
---

# You had only one Job!

- ...

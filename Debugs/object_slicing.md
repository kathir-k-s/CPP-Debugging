# Problem Title

## Description/Code for the Erroneous Code

```cpp
#include<iostream>
#include <memory>

class Vehicle
{
public:
    Vehicle(const std::string& strName)
        :m_strName(strName)
    {
    }

    virtual ~Vehicle() {}

    void SetName(const std::string& strName)
    {
        m_strName = strName;
    }

    const std::string GetName() const
    {
        return m_strName;
    }

private:
    std::string m_strName;
};

class Car : public Vehicle
{
public:
    Car(const std::string& strName, const double& dAirbagSize)
        :Vehicle(strName), m_dAirBagSize(dAirbagSize)
    {
    }

    void SetAirBagSize(const double& dAirbagSize)
    {
        m_dAirBagSize = dAirbagSize;
    }

    const double GetAirBagSize() const
    {
        return m_dAirBagSize;
    }

private:
    double m_dAirBagSize;
};


int main()
{
    std::unique_ptr<Vehicle> cVehicleA;
    std::unique_ptr<Vehicle> cVehicleB;

    cVehicleA = std::unique_ptr<Vehicle>(new Vehicle("Maserati"));
    cVehicleB = std::unique_ptr<Vehicle>(new Car("Beetle", 25.0));

    std::cout << cVehicleA->GetName() << std::endl;
    std::cout << cVehicleB->GetName() << std::endl;

    Vehicle cVehicleG = *cVehicleB.get();
    Car* cCarB = static_cast<Car*>(&cVehicleG);
    std::cout << cCarB->GetAirBagSize() << std::endl;
}
````
```output
Maserati
Beetle
1.30017e-311
```
- `cCarB->GetAirBagSize()` is returning `garbase`

## Root Cause 
- When `*cVehicleB.get()` is assigned to `cVehicleG`. `Car` properties (like `m_dAirBagSize`) is sliced from `cVehicleG`.
- This is because `cVehicleG` only allocates memory for `Vechicle`. so it doesnot have space for `airbag`. hence properties of `Car` is left out.
- So when accessing the `airbag` after downcasting it gives `garbage` value. but it prints `name` correctly.

## Solution
```cpp
int main()
{
    std::unique_ptr<Vehicle> cVehicleA;
    std::unique_ptr<Vehicle> cVehicleB;

    cVehicleA = std::unique_ptr<Vehicle>(new Vehicle("Maserati"));
    cVehicleB = std::unique_ptr<Vehicle>(new Car("Beetle", 25.0));

    std::cout << cVehicleA->GetName() << std::endl;
    std::cout << cVehicleB->GetName() << std::endl;

    Vehicle* cVehicleG = cVehicleB.get();
    Car* cCarB = static_cast<Car*>(cVehicleG);
    std::cout << cCarB->GetAirBagSize() << std::endl;
}

```
```output
Maserati
Beetle
25
```
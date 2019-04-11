# typeClassesPoC

This repository consists of a proof of concept about Type Classes for learning and 
understanding. Also, it includes an explanation below

## What are the Type Classes?

At first, we need to understand what is Polymorphism. Polymorphism is an useful feature 
in high level languages. It allows us to create interfaces, operate with abstraction and 
deal with implementation based on types.

Most OOP languages uses inheritance to achive polymorphism. But, there is another way to achive 
polymorphism: **type classes**.

**TL;DR**: OOP puts data and function in the one place (classes). Type classes approach is different,
entities are data, and they are declouped from entities responsible for implementation.

_Ok, but show me the code..._

Let's try to understand type classes step by step, from an OOP polymorphism example to type classes.

### Inheritance fashion

We have a notion of a vehicle that has some weight and multiple implementations.

```scala
// Our generic inteface
trait Vehicle {
  def weight: Int
}

// Implementations
class Car(passengers: Int) extends Vehicle {
  override def weight: Int = passengers * 75
}

class Truck(passengers: Int, load: Int) extends Vehicle {
  override def weight: Int = (passengers * 75) + length
}

// Generic function
def weightOf(vehicle: Vehicle): Int = vehicle.weight

// Usage
areaOf(new Car(5))
areaOf(new Truck(1, 5000))
```

Our generic function works on vehicles because we pass it the implementation excplicitly.

### Type class fashion

Firstly, we are going to introduce a new class to avoid extending our Car or Truck from Vehicle.

```scala
// Our generic inteface
trait Vehicle {
  def weight: Int
}

// Shape definition data structures, should be in a diffrent file/namespace
case class Car(passengers: Int)
case class Truck(passengers: Int, load: Int)

// Implementations
class CarVehicle(passengers: Int) extends Vehicle {
  override def weight: Int = passengers * 75
}

class TruckVehicle(passengers: Int, load: Int) extends Vehicle {
  override def weight: Int = (passengers * 75) + length
}

// Generic function
def weightOf(vehicle: Vehicle): Int = vehicle.weight

// Usage
areaOf(new CarVehicle(5))
areaOf(new TruckVehicle(1, 5000))
```

We added two case classes, car and truck, which have the data. In other place, we have two 
weight function implementation: `CarVehicle` and `TruckVehicle`. Here, we have separated data 
from functionality.

Now, we don't achieve anything and we have two problems:

1. Code duplication because if we change `Car` we need to change `CarVehicle`.
2. `weightOf` function needs the instance, not a case class itself. 

Let's go to fix these removing constructors:

```scala
// Our generic inteface
trait Vehicle {
  def weight: Int
}

// Shape definition data structures, should be in a diffrent file/namespace
case class Car(passengers: Int)
case class Truck(passengers: Int, load: Int)

// Implementations
class CarVehicle extends Vehicle {
  override def weight: Int = ???
}

class TruckVehicle extends Vehicle {
  override def weight: Int = ???
}

// Generic function
def weightOf(vehicle: Vehicle): Int = vehicle.weight

// Usage
areaOf(new CarVehicle)
areaOf(new TruckVehicle)
```

No code duplication, but we need the information about in order to calculate the weight.
We need the case class.

```scala
trait Vehicle {
  def weight(???): Int //Won't compile
}

case class Car(passengers: Int)
case class Truck(passengers: Int, load: Int)

class CarVehicle extends Vehicle {
  override def weight(car: Car): Int = car.passengers * 75
}

class TruckVehicle extends Vehicle {
  override def weight(truck: Truck): Int = (truck.passengers * 75) + truck.load
}

def weightOf(vehicle: Vehicle): Int = vehicle.weight(???)
```

Now, we are going to add generics or type parameter:

```scala
trait Vehicle[A] {
  def weight(a: A): Int
}

case class Car(passengers: Int)
case class Truck(passengers: Int, load: Int)

class CarVehicle extends Vehicle[Car] {
  override def weight(car: Car): Int = car.passengers * 75
}

class TruckVehicle extends Vehicle[Truck] {
  override def weight(truck: Truck): Int = (truck.passengers * 75) + truck.load
}

def weightOf[A](vehicle: Vehicle[A]): Int = vehicle.weight(???)

areaOf(new CarVehicle)
areaOf(new TruckVehicle)
```

Now, we need to pass the implementation to call `weightOf` function. 

```scala
trait Vehicle[A] {
  def weight(a: A): Int
}

case class Car(passengers: Int)
case class Truck(passengers: Int, load: Int)

class CarVehicle extends Vehicle[Car] {
  override def weight(car: Car): Int = car.passengers * 75
}

class TruckVehicle extends Vehicle[Truck] {
  override def weight(truck: Truck): Int = (truck.passengers * 75) + truck.load
}

def weightOf[A](vehicleInfo: A, vehicle: Vehicle[A]): Int = vehicle.weight(vehicleInfo)

areaOf(Car(5), new CarVehicle)
areaOf(Truck(1,5000), new TruckVehicle)
```

Let's rename the params to make more sense:

```scala
def weightOf[A](vehicle: A, vehicleImpl: Vehicle[A]): Int = vehicleImpl.weight(vehicle)

areaOf(Car(5), new CarVehicle)
areaOf(Truck(1,5000), new TruckVehicle)
```

We have to pass the implementation explicitly, but in Scala we can pass the implementation by implicity.

```Scala
def weightOf[A](vehicle: A)(implicit vehicleImpl: Vehicle[A]): Int = vehicleImpl.weight(vehicle)

areaOf(Car(5))
areaOf(Truck(1,5000)
```

This won't compile, we need to declare our implementations as implicit:

```scala
implicit val carVehicle = new CarVehicle
implicit val truckVehicle = new TruckVehicle

areaOf(Car(5))
areaOf(Truck(1,5000)
```

It feels like a boilerplate, we have to instantiate them or import. We are going to fix that:

```scala
trait Vehicle[A] {
 def weight(a: A): Int
}

case class Car(passengers: Int)
case class Truck(passengers: Int, load: Int)

implicit object CarVehicle extends Vehicle[Car] {
 override def weight(car: Car): Int = car.passengers * 75
}

implicit object TruckVehicle extends Vehicle[Truck] {
 override def weight(truck: Truck): Int = (truck.passengers * 75) + truck.load
}

def weightOf[A](vehicle: A)(implicit vehicleImpl: Vehicle[A]): Int = vehicleImpl.weight(vehicle)

areaOf(Car(5))
areaOf(Truck(1,5000)
``` 

And, that's all folks!

## Code

### Building

_Under construction_

### Running

_Under construction_

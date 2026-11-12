# Modèle de données - Plateforme de covoiturage.


Regional carpooling platform (Symfony 7.4 / MySQL 8 / Doctrine ORM).

This document contains:

1. the use case diagram
2. the class diagram

---


## 1. Use case diagram

```mermaid
flowchart LR
  V(["👤 Visitor"])
  P(["🧍 Passenger"])
  D(["🚗 Driver"])
  A(["🛠️ Administrator"])

  subgraph SYS["Regional carpooling platform"]
    direction TB

    subgraph PUB["Public area"]
      direction TB
      U1(["Register / Log in"])
      U2(["Search trips<br/>(departure, arrival, date)"])
      U3(["View trip details and map"])
      U4(["View profile and reviews"])
    end

    subgraph PAS["Passenger area"]
      direction TB
      P1(["Book all or part of a trip"])
      P2(["Pay (simulated)"])
      P3(["Cancel my booking"])
      P4(["View my bookings"])
      P5(["Review the driver"])
    end

    subgraph DRV["Driver area"]
      direction TB
      D1(["Manage my vehicles"])
      D2(["Publish a trip"])
      D3(["Get suggested price"])
      D4(["Edit / cancel a trip"])
      D5(["View passengers"])
      D6(["Mark trip as completed"])
      D7(["Review a passenger"])
      D8(["Driver dashboard / history"])
    end

    subgraph ADM["Back-office"]
      direction TB
      A1(["Manage users, cities,<br/>vehicles, trips, bookings"])
      A2(["Moderate reviews"])
      A3(["View statistics"])
    end

    P1 -. include .-> P2
    D2 -. include .-> D3
    D4 -. include .-> P3
    P5 -. "extend (trip completed)" .-> P4
    U3 -. "extend" .-> D3
  end

  subgraph EXT["External services"]
    direction TB
    NOM(["🌍 Nominatim<br/>(geocoding)"])
    OSRM(["🛣️ OSRM<br/>(route, distance)"])
    PAY(["💳 Payment simulator"])
  end

  V --> PUB
  P --> PUB
  P --> PAS
  D --> PUB
  D --> DRV
  D --> PAS
  A --> ADM

  D3 --> NOM
  D3 --> OSRM
  U3 --> OSRM
  P2 --> PAY
```

---

## 2. Class diagram

```mermaid
classDiagram
direction TB

namespace Users_and_Vehicles {
  class User {
    +int id
    +string email
    +string password
    +string firstName
    +string lastName
    +string phone
    +array roles
    +datetime createdAt
  }
  class Vehicle {
    +int id
    +string brand
    +string model
    +string color
    +string plateNumber
    +int seats
  }
}

namespace Geography {
  class City {
    +int id
    +string name
    +string region
    +float latitude
    +float longitude
  }
}

namespace Trips {
  class Trip {
    +int id
    +datetime departureAt
    +int totalSeats
    +decimal pricePerKm
    +decimal suggestedPrice
    +float distanceKm
    +text routeGeometry
    +TripStatus status
    +int version
    +datetime completedAt
    +datetime cancelledAt
    +availableSeats(from, to) int
    +segmentLoad(from, to) Map~segment, bookedSeats~
    +cancel(reason)
    +complete()
  }
  class TripStop {
    +int id
    +int position
    +datetime arrivalAt
    +datetime departureAt
    +float distanceFromStartKm
  }
  class SegmentOccupancy {
    +int id
    +int fromPosition
    +int toPosition
    +int bookedSeats
    +int version
    +add(seats)
    +remove(seats)
  }
}

namespace Reservations {
  class Booking {
    +int id
    +int seats
    +decimal totalPrice
    +BookingStatus status
    +datetime createdAt
    +datetime cancelledAt
    +string cancellationReason
    +string idempotencyKey
    +cancelByPassenger()
    +cancelByDriver()
    +complete()
    +canBeReviewed() bool
  }
  class Payment {
    +int id
    +decimal amount
    +decimal refundedAmount
    +PaymentStatus status
    +string simulatedReference
  }
}

namespace Feedback {
  class Review {
    +int id
    +int rating
    +text comment
    +ReviewDirection direction
    +datetime createdAt
  }
}

namespace Enumerations {
  class TripStatus {
    <<enumeration>>
    PUBLISHED
    IN_PROGRESS
    COMPLETED
    CANCELLED
  }
  class BookingStatus {
    <<enumeration>>
    PENDING
    CONFIRMED
    CANCELLED_BY_PASSENGER
    CANCELLED_BY_DRIVER
    COMPLETED
  }
  class PaymentStatus {
    <<enumeration>>
    PAID
    REFUNDED
    PARTIALLY_REFUNDED
  }
  class ReviewDirection {
    <<enumeration>>
    PASSENGER_TO_DRIVER
    DRIVER_TO_PASSENGER
  }
}

User "1" --> "*" Vehicle : owns
User "1" --> "*" Trip : drives
Vehicle "1" --> "*" Trip : used for
Trip "1" *-- "2..*" TripStop : stops
TripStop "*" --> "1" City : at
Trip "1" *-- "*" SegmentOccupancy : tracks
Trip "1" --> "*" Booking : has
User "1" --> "*" Booking : books
Booking "*" --> "1" TripStop : boards at
Booking "*" --> "1" TripStop : alights at
Booking "1" *-- "1" Payment : paid by
Booking "1" --> "0..2" Review : leads to
User "1" --> "*" Review : writes
User "1" --> "*" Review : receives

Trip ..> TripStatus
Booking ..> BookingStatus
Payment ..> PaymentStatus
Review ..> ReviewDirection
```

---
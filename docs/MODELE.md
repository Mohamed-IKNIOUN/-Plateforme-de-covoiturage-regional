# Modèle de données - Plateforme de covoiturage.


Regional carpooling platform (Symfony 7.4 / MySQL 8 / Doctrine ORM).

This document contains:

1. the use case diagram

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


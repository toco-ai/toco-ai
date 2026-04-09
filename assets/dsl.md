# Toco Design Element DSL Reference

This document extracts the DSL expressions for each design element from the Toco platform knowledge base, for quick reference during design, review, code generation, and troubleshooting.

Each design element contains three sections:
- **Purpose**: what problem this element solves and which layer it typically belongs to
- **DSL Example**: how to express it
- **Generated Code**: which code files are produced and their responsibilities

## Table of Contents
- [1. Enum](#1-enum)
- [2. Value Object (EO)](#2-value-object-eo)
- [3. Entity](#3-entity)
- [4. Aggregate Object (BO)](#4-aggregate-object-bo)
- [5. Data Transfer Object (DTO)](#5-data-transfer-object-dto)
- [6. View Object (VO)](#6-view-object-vo)
- [7. Read Plan (ReadPlan)](#7-read-plan-readplan)
- [8. Write Plan (WritePlan)](#8-write-plan-writeplan)
- [9. Service Layer Method (RPC / SERVICE)](#9-service-layer-method-rpc--service)
- [10. API](#10-api)
- [11. Flow Service (FunctionFlow)](#11-flow-service-functionflow)
- [12. Domain Message (DomainMessage)](#12-domain-message-domainmessage)
- [13. Common Message (CommonMessage)](#13-common-message-commonmessage)
- [14. Subscribe Message (SubscribeMessage)](#14-subscribe-message-subscribemessage)

## 1. Enum

### Purpose
Enum expresses a fixed set of constants, and can be used as the field type for Entity, DTO, VO, EO, and other structures.

### DSL Example
```json
{
  "name": "room_status_enum",
  "uuid": null,
  "description": "Room status",
  "moduleName": "room",
  "values": [
    "AVAILABLE",
    "OCCUPIED",
    "DISABLED"
  ]
}
```

### Generated Code
| File | Description |
| --- | --- |
| `RoomStatusEnum` | Enum definition class expressing the constant set; should not be modified manually |

## 2. Value Object (EO)

### Purpose
EO is a Value Object in DDD, typically used to express reusable composite structures. It can be used as a field type for Entity, DTO, VO, BTO, and QTO.

### DSL Example
```json
{
  "name": "address_eo",
  "description": "Address value object",
  "uuid": null,
  "moduleName": "user",
  "fieldList": [
    {
      "name": "province",
      "type": "String"
    },
    {
      "name": "city",
      "type": "String"
    },
    {
      "name": "detail_address",
      "type": "String"
    }
  ]
}
```

### Generated Code
| File | Description |
| --- | --- |
| `AddressEo` | EO structure definition class, expressing a composite value object |

## 3. Entity

### Purpose
Entity corresponds to a database table and is the core design element for ER modeling. It defines fields, primary keys, indexes, and other database structure information.

### DSL Example
```text
Entity room {
  name: "room"
  description: "room entity, basic room information"
  primaryFieldName: "room_id"

  fieldList: [
    {
      name: "room_id"
      type: "Long"
      allowNull: false
    },
    {
      name: "room_number"
      type: "String"
      length: 50
      allowNull: false
    },
    {
      name: "room_status"
      type: "Enum"
      typeObject: "room_status_enum"
      allowNull: false
      defaultValue: "AVAILABLE"
    },
    {
      name: "room_type_id"
      type: "Long"
      foreignKey: "room_type"
      foreignKeyType: "many"
      allowNull: false
    }
  ]

  indexList: [
    {
      name: "PRIMARY"
      fieldNameList: ["room_id"]
      unique: true
    },
    {
      name: "IDX_ROOM_NUMBER"
      fieldNameList: ["room_number"]
      unique: true
    }
  ]
}
```

### Generated Code
| File | Description |
| --- | --- |
| `Room` | DO/Entity structure definition class, corresponding to the database table |
| `RoomMapper` | MyBatis-Plus Mapper |
| `RoomDao` | Dao interface |
| `RoomDaoImpl` | Dao implementation |

## 4. Aggregate Object (BO)

### Purpose
BO expresses the aggregate boundary, organizing the aggregate root entity and sub-entities into a tree structure. It is the foundation for write chains and business invariant validation.

### DSL Example
```json
{
  "bo": {
    "name": "booking_order_bo",
    "rootEntity": "booking_order",
    "children": [
      {
        "entity": "booking_order_item",
        "fieldName": "booking_order_item_list",
        "relationType": "1:N"
      }
    ]
  }
}
```

### Generated Code
| File | Description |
| --- | --- |
| `BookingOrderBO` | Aggregate object; validation logic can be extended here |
| `BookingOrderBaseBO` | Aggregate base class, carrying generated structure and relationship code |

## 5. Data Transfer Object (DTO)

### Purpose
DTO carries data across service layers. It inherits fields from an entity, expands nested DTOs via foreign keys, and allows a small number of custom fields. DTO must not be used directly as an HTTP response value.

### DSL Example
```json
{
  "dto": {
    "uuid": null,
    "name": "room_with_type_dto",
    "description": "Room detail including room type",
    "fromEntity": "room",
    "expandList": [
      {
        "foreignKeyInThisEntity": "room_type_id",
        "dtoFieldName": "room_type",
        "dto": {
          "uuid": "room-type-base-dto-uuid",
          "name": "room_type_base_dto",
          "fromEntity": "room_type",
          "description": "Basic room type info"
        }
      }
    ],
    "customFieldList": [
      {
        "name": "available_count",
        "type": "Integer",
        "description": "Available room count"
      }
    ]
  }
}
```

### Generated Code
| File | Description |
| --- | --- |
| `RoomWithTypeDto` | DTO structure definition |
| `RoomWithTypeDtoManager` | DTO Manager interface |
| `RoomWithTypeDtoManagerImpl` | DTO Manager implementation |
| `RoomWithTypeDtoConverter` | Converter from Entity/BaseDTO to DTO |
| `RoomWithTypeDtoService` | Preset query service for DTO |
| `RoomWithTypeDtoDataAssembler` | DTO data assembly and custom field extension point |
| `RoomWithTypeDtoBaseDataAssembler` | DTO DataAssembler base class |

## 6. View Object (VO)

### Purpose
VO is used for API responses. It is typically derived from a DTO, can trim fields, expand nested VO structures, and add display-layer custom fields.

### DSL Example
```json
{
  "vo": {
    "uuid": null,
    "name": "room_list_vo",
    "description": "Room list display object",
    "rootVo": "room_list_vo",
    "moduleName": "room",
    "fromEntity": "room",
    "fromDto": "room_with_type_dto",
    "fromDtoUuid": "room-with-type-dto-uuid",
    "extendFieldList": [
      {
        "name": "room_id"
      },
      {
        "name": "room_number"
      }
    ],
    "expandList": [
      {
        "foreignKeyInThisEntity": "room_type_id",
        "voFieldName": "room_type",
        "vo": {
          "name": "room_type_vo",
          "description": "Room type info",
          "rootVo": "room_list_vo",
          "fromEntity": "room_type",
          "fromDto": "room_type_base_dto",
          "fromDtoUuid": "room-type-base-dto-uuid",
          "extendFieldList": [
            {
              "name": "room_type_id"
            },
            {
              "name": "description"
            }
          ]
        }
      }
    ],
    "customFieldList": []
  }
}
```

### Generated Code
| File | Description |
| --- | --- |
| `RoomListVo` | VO structure definition for API responses |
| `RoomListVoConverter` | Converter from DTO to VO |
| `RoomListVoDataAssembler` | VO data assembly and display extension point |
| `RoomListVoBaseDataAssembler` | VO DataAssembler base class |

## 7. Read Plan (ReadPlan)

### Purpose
ReadPlan defines complex query logic. It can return DTO or VO, and supports pagination, count, sorting, and field filtering.

### DSL Example
```json
{
  "name": "query_available_rooms_excluding_booked",
  "uuid": null,
  "woId": "room-with-booking-wo-uuid",
  "description": "Query available rooms, excluding those occupied during the given time range",
  "paginationTypes": ["pagination", "count"],
  "query": "room_type.capacity >= #capacityGte AND NOT ( booking_order_item_list contains ( check_in_date < #checkOutDate AND check_out_date > #checkInDate ) )",
  "voOrDtoId": "room-list-vo-uuid",
  "defaultOrder": [
    {
      "fieldPath": "room_id",
      "direction": "ASC"
    }
  ],
  "filters": []
}
```

### Generated Code
| File | Description |
| --- | --- |
| `QueryAvailableRoomsExcludingBookedQto` | Query parameter object |
| `QueryAvailableRoomsExcludingBookedQtoDao` | Read plan Dao |
| `QueryAvailableRoomsExcludingBookedQueryHandler` | Query entry point, returning DTO or VO |

## 8. Write Plan (WritePlan)

### Purpose
WritePlan is the unified entry point for database write operations. It organizes create, update, delete, and batch-merge operations within the aggregate boundary.

### DSL Example
```json
{
  "uuid": null,
  "name": "create_order_with_items",
  "description": "Create order with line items",
  "bo": "booking_order",
  "operations": [
    {
      "bo": "booking_order",
      "action": "CREATE",
      "fields": [
        "order_number",
        "member_id",
        "order_time",
        "order_status",
        "total_amount"
      ]
    },
    {
      "bo": "booking_order_item",
      "action": "CREATE",
      "fields": [
        "room_id",
        "check_in_date",
        "check_out_date",
        "actual_amount",
        "item_status"
      ]
    }
  ]
}
```

### Generated Code
| File | Description |
| --- | --- |
| `CreateOrderWithItemsBto` | Write plan parameter object |
| `BookingOrderBOService` | Service entry method for the write plan |
| `BaseBookingOrderBOService` | Aggregate write base implementation, responsible for actual aggregate write operations |

## 9. Service Layer Method (RPC / SERVICE)

### Purpose
Service layer methods are divided into two types:
- `SERVICE`: service capability available within the module
- `RPC`: cross-module capability exposed publicly

They differ only in visibility; the method signature DSL structure is essentially the same.

### DSL Example
```json
{
  "className": "RoomPriceService",
  "methodName": "calculateTotalPrice",
  "type": "SERVICE",
  "requestParams": [
    {
      "name": "roomTypeId",
      "description": "Room type ID",
      "type": "Long"
    },
    {
      "name": "checkInDate",
      "description": "Check-in date",
      "type": "Date"
    },
    {
      "name": "checkOutDate",
      "description": "Check-out date",
      "type": "Date"
    }
  ],
  "response": {
    "type": "BigDecimal"
  }
}
```

To make it an RPC, simply change `type` to `"RPC"`.

### Generated Code
| File | Description |
| --- | --- |
| `RoomPriceService` | Service class and corresponding method implementation |
| `PubRoomPriceService` | Public interface generated when the method is RPC and subscribed by other modules |

## 10. API

### Purpose
API defines the outward-facing HTTP interface, specifying the URI, request method, parameter structure, and response structure contract.

### DSL Example
```json
{
  "methodName": "createMeeting",
  "className": "MeetingManageController",
  "uri": "/api/meeting/create-meeting",
  "requestMethod": "POST",
  "contentType": "JSON",
  "usageScenario": "Create a meeting",
  "requestParams": [
    {
      "name": "createMeetingBto",
      "description": "Create meeting parameters",
      "type": "Bto",
      "typeUuid": "create-meeting-write-plan-uuid"
    },
    {
      "name": "test",
      "description": "Whether this is a test request",
      "type": "Boolean"
    }
  ],
  "response": {
    "type": "Vo",
    "typeUuid": "meeting-detail-vo-uuid"
  }
}
```

### Generated Code
| File | Description |
| --- | --- |
| `MeetingManageController` | Controller class and API method |
| `MeetingDetailVo` | Response VO for the API |
| `${MethodName}Request` (auto-wrapped or generated) | When POST + JSON with multiple body params, automatically wraps them into a request body object |

## 11. Flow Service (FunctionFlow)

### Purpose
FunctionFlow expresses complex business flows by decomposing business logic into start nodes, sequential nodes, condition nodes, and selection nodes. The platform generates the flow skeleton code.

### DSL Example
```json
{
  "name": "user_register",
  "moduleName": "user",
  "uuid": null,
  "description": "User registration flow",
  "nodes": [
    {
      "name": "start",
      "type": "START_NODE",
      "description": "Start node"
    },
    {
      "name": "check_user_registered",
      "type": "CONDITION_NODE",
      "description": "Check if user is already registered"
    },
    {
      "name": "create_user",
      "type": "PROCESS_NODE",
      "description": "Create user and send notification"
    },
    {
      "name": "return_user_info",
      "type": "PROCESS_NODE",
      "description": "Return user info"
    }
  ],
  "edges": [
    {
      "fromNode": "start",
      "toNode": "check_user_registered"
    },
    {
      "fromNode": "check_user_registered",
      "toNode": "create_user",
      "value": false
    },
    {
      "fromNode": "check_user_registered",
      "toNode": "return_user_info",
      "value": true
    },
    {
      "fromNode": "create_user",
      "toNode": "return_user_info"
    }
  ]
}
```

### Generated Code
| File | Description |
| --- | --- |
| `UserFlowConfig` | Flow registration configuration |
| `UserFlowService` | Flow entry service |
| `UserRegisterContext` | Flow context object |
| `CreateUserNode` | Node handler class |

## 12. Domain Message (DomainMessage)

### Purpose
Domain messages listen to create, update, or delete events on an entity within an aggregate. The message payload is automatically generated by the platform, and the send logic is automatically triggered when entity state changes.

### DSL Example
```json
{
  "name": "user_created",
  "description": "User successfully created message",
  "bo": "user",
  "entity": "user",
  "fields": [
    "user_id",
    "user_name"
  ],
  "action": "create",
  "delayInSeconds": 0,
  "uuid": null
}
```

### Generated Code
| File | Description |
| --- | --- |
| `UserCreatedMo` | Domain message payload class |
| Hibernate listener generation logic | Automatically sends the domain message on entity change; no manual send code needed |

## 13. Common Message (CommonMessage)

### Purpose
Common messages define an actively sendable message payload that does not depend on entity state changes. Suitable for async notifications, cross-module decoupling, and delayed tasks.

### DSL Example
```json
{
  "name": "user_registered",
  "description": "User registration success notification",
  "uuid": null,
  "moduleName": "user",
  "delay": false,
  "transactional": true,
  "fieldList": [
    {
      "name": "user_id",
      "type": "Long",
      "description": "User ID"
    },
    {
      "name": "nick_name",
      "type": "String",
      "description": "Nickname"
    }
  ]
}
```

### Generated Code
| File | Description |
| --- | --- |
| `UserRegisteredMo` | Common message payload class |
| `UserRegisteredMoMessageSender` | Message sender entry class |

## 14. Subscribe Message (SubscribeMessage)

### Purpose
Subscribe messages subscribe a domain message or common message to a specified module. The platform automatically generates consumer template code; developers only need to fill in the consumption logic.

### DSL Example
```json
{
  "msgName": "user_registered",
  "moduleName": "coupon"
}
```

Or subscribe by message ID:

```json
{
  "msgId": "user-registered-msg-uuid",
  "moduleName": "coupon"
}
```

### Generated Code
| File | Description |
| --- | --- |
| `UserRegisteredConsumer` | Message consumer class |
| `handleMessage(...)` | Consumer entry method; developers add business logic here |

## Notes

### 1. Generated artifacts that should not be modified directly
The following types should be regenerated by modifying the design element, rather than editing the generated file directly:
- Entity / DO
- Enum / EO
- DTO / VO structure definitions
- Mapper / Dao base code
- BaseBO / BaseDataAssembler / BaseQueryHandler base classes

### 2. Extension points suitable for manual logic
- `BO`: `validateAggregate` / `valid`
- `DtoDataAssembler#postProcessData`
- `VoDataAssembler#postProcessData`
- `QueryHandler`
- `Controller`
- Message `Consumer#handleMessage`

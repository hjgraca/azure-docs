---
title: Azure Blockchain Workbench configuration reference
description: Azure Blockchain Workbench application configuration overview.
services: azure-blockchain
keywords: 
author: PatAltimore
ms.author: patricka
ms.date: 3/20/2018
ms.topic: article
ms.service: azure-blockchain
ms.reviewer: zeyadr
manager: femila
---
# Azure Blockchain Workbench configuration reference

 Azure Blockchain Workbench applications are multi-party workflows defined by configuration metadata and smart contract code. Configuration metadata defines the high-level workflows and interaction model of the blockchain application. Smart contracts define the business logic of the blockchain application. Workbench uses configuration and smart contract code to generate  blockchain application user experiences.

Configuration metadata specifies the following information for each blockchain application: 

* Name and description of the blockchain application
* Unique roles for users who can act or participate within the blockchain application
* One or more workflows. Each workflow acts as a state machine to control the flow of the business logic. Workflows can be independent or interact with one another.

Each defined workflow specifies the following:

* Name and description of the workflow
* States of the workflow.  Each state is a stage in the business logic's control flow. 
* Actions to transition to the next state
* User roles permitted to initiate each action
* Smart contracts that represent business logic in code files

## Application

A blockchain application contains configuration metadata, workflows, and user roles who can act or participate within the application.

| Field | Description | Required |
|-------|-------------|:--------:|
| ApplicationName | Unique application name. The corresponding smart contract must use the same **ApplicationName** for the applicable contract class.  | Yes |
| DisplayName | Friendly display name of the application. | Yes |
| Description | Description of the application. | No |
| ApplicationRoles | Collection of [ApplicationRoles](#application-roles). User roles who can act or participate within the application.  | Yes |
| Workflows | Collection of  [Workflows](#workflows). Each workflow acts as a state machine to control the flow of the business logic. | Yes |

For an example, see [configuration file example](#configuration-file-example).

## Workflows

An application's business logic may be modeled as a state machine where taking an action causes the flow of the business logic to move from one state to another. A workflow is a collection of such states and actions. Each workflow consists of one or more smart contracts, which represent the business logic in code files. An executable contract is an instance of a workflow.

| Field | Description | Required |
|-------|-------------|:--------:|
| Name | Unique workflow name. The corresponding smart contract must use the same **Name** for the applicable contract class. | Yes |
| DisplayName | Friendly display name of the workflow. | Yes |
| Description | Description of the workflow. | No |
| Initiators | Collection of [ApplicationRoles](#application-roles). Roles that are assigned to users who are authorized to create contracts in the workflow. | Yes |
| StartState | Name of the initial state of the workflow. | Yes |
| Properties | Collection of [identifiers](#identifiers). Represents data that can be read off-chain or visualized in a user experience tool. | Yes |
| Constructor | Defines input parameters for creating an instance of the workflow. | Yes |
| Functions | A collection of [functions](#functions) that can be executed in the workflow. | Yes |
| States | A collection of workflow [states](#states). | Yes |

For an example, see [configuration file example](#configuration-file-example).

## Type

Supported data types.

| Type | Description |
|-------|-------------|
| address  | Blockchain address type, such as *contracts* or *users* |
| bool     | Boolean data type |
| contract | Address of type contract |
| int      | Integer data type |
| money    | Money data type |
| state    | Workflow state |
| string   | String data type |
| user     | Address of type user |
| time     | Time data type |
|`[ Application Role Name ]`| Any name specified in application role. Limits users to be of that role type. |

## Constructor

Defines input parameters for an instance of a workflow.

| Field | Description | Required |
|-------|-------------|:--------:|
| Parameters | Collection of [identifiers](#identifiers) required to initiate a smart contract. | Yes |

### Constructor example

``` json
{
  "Parameters": [
    {
      "Name": "description",
      "Description": "The description of this asset",
      "DisplayName": "Description",
      "Type": {
        "Name": "string"
      }
    },
    {
      "Name": "price",
      "Description": "The price of this asset",
      "DisplayName": "Price",
      "Type": {
        "Name": "money"
      }
    }
  ]
}

```

## Functions

Defines functions that can be executed on the workflow.

| Field | Description | Required |
|-------|-------------|:--------:|
| Name | The unique name of the function. The corresponding smart contract must use the same **Name** for the applicable function. | Yes |
| DisplayName | Friendly display name of the function. | Yes |
| Description | Description of the function | No |
| Parameters | Collection of [identifiers](#identifiers) corresponding to the parameters of the function. | Yes |

### Functions example

``` json
"Functions": [
  {
    "Name": "Modify",
    "DisplayName": "Modify",
    "Description": "Modify the description/price attributes of this asset transfer instance",
    "Parameters": [
      {
        "Name": "description",
        "Description": "The new description of the asset",
        "DisplayName": "Description",
        "Type": {
          "Name": "string"
        }
      },
      {
        "Name": "price",
        "Description": "The new price of the asset",
        "DisplayName": "Price",
        "Type": {
          "Name": "money"
        }
      }
    ]
  },
  {
    "Name": "Terminate",
    "DisplayName": "Terminate",
    "Description": "Used to cancel this particular instance of asset transfer",
    "Parameters": []
  }
]

```

## States

A collection of unique states within a workflow. Each state captures a step in the business logic's control flow. 

| Field | Description | Required |
|-------|-------------|:--------:|
| Name | Unique name of the state. The corresponding smart contract must use the same **Name** for the applicable state. | Yes |
| DisplayName | Friendly display name of the state. | Yes |
| Description | Description of the state. | No |
| PercentComplete | An integer value displayed in the Blockchain Workbench user interface to show the progress within the business logic control flow. | Yes |
| Style | Visual hint indicating whether the state represents a success or failure state. There are two valid values: `Success` or `Failure`. | Yes |
| Transitions | Collection of available [transitions](#transitions) from the current state to the next set of states. | No |

### States example

``` json
"States": [
    {
      "Name": "Active",
      "DisplayName": "Active",
      "Description": "The initial state of the asset transfer workflow",
      "PercentComplete": 20,
      "Style": "Success",
      "Transitions": [
        {
          "AllowedRoles": [],
          "AllowedInstanceRoles": [ "InstanceOwner" ],
          "Description": "Cancels this instance of asset transfer",
          "Function": "Terminate",
          "NextStates": [ "Terminated" ],
          "DisplayName": "Terminate Offer"
        },
        {
          "AllowedRoles": [ "Buyer" ],
          "AllowedInstanceRoles": [],
          "Description": "Make an offer for this asset",
          "Function": "MakeOffer",
          "NextStates": [ "OfferPlaced" ],
          "DisplayName": "Make Offer"
        },
        {
          "AllowedRoles": [],
          "AllowedInstanceRoles": [ "InstanceOwner" ],
          "Description": "Modify attributes of this asset transfer instance",
          "Function": "Modify",
          "NextStates": [ "Active" ],
          "DisplayName": "Modify"
        }
      ]
    },
    {
      "Name": "Accepted",
      "DisplayName": "Accepted",
      "Description": "Asset transfer process is complete",
      "PercentComplete": 100,
      "Style": "Success",
      "Transitions": []
    },
    {
      "Name": "Terminated",
      "DisplayName": "Terminated",
      "Description": "Asset transfer has been cancelled",
      "PercentComplete": 100,
      "Style": "Failure",
      "Transitions": []
    }
  ]
```

## Transitions

Available actions to the next state. One or more user roles may perform an action at each state, where an action may transition a state to another state in the workflow. 

| Field | Description | Required |
|-------|-------------|:--------:|
| AllowedRoles | List of applications roles allowed to initiate the transition. All users of the specified role may be able to perform the action. | No |
| AllowedInstanceRoles | List of user roles participating or specified in the smart contract allowed to initiate the transition. Instance roles are defined in **Properties** within Workflows. These users represent a user participating or specified in the smart contract as opposed to all users of a role type. | No |
| DisplayName | Friendly display name of the transition. | Yes |
| Description | Description of the transition. | No |
| Function | The name of the function to initiate the transition. | Yes |
| NextStates | A collection of potential next states after a successful transition. | Yes |

### Transitions example

``` json
"Transitions": [
  {
    "AllowedRoles": [],
    "AllowedInstanceRoles": [ "InstanceOwner" ],
    "Description": "Cancels this instance of asset transfer",
    "Function": "Terminate",
    "NextStates": [ "Terminated" ],
    "DisplayName": "Terminate Offer"
  },
  {
    "AllowedRoles": [ "Buyer" ],
    "AllowedInstanceRoles": [],
    "Description": "Make an offer for this asset",
    "Function": "MakeOffer",
    "NextStates": [ "OfferPlaced" ],
    "DisplayName": "Make Offer"
  },
  {
    "AllowedRoles": [],
    "AllowedInstanceRoles": [ "InstanceOwner" ],
    "Description": "Modify attributes of this asset transfer instance",
    "Function": "Modify",
    "NextStates": [ "Active" ],
    "DisplayName": "Modify"
  }
]

```

## Application roles

Application roles define a set of roles that can be assigned to users who want to act or participate within the application. Application roles can be used to restrict actions and participation within the blockchain application and corresponding workflows. 

| Field | Description | Required |
|-------|-------------|:--------:|
| Name | The unique name of the application role. The corresponding smart contract must use the same **Name** for the applicable role. Base type names are reserved. You cannot name an application role with the same name as [Type](#type)| Yes |
| Description | Description of the application role. | No |

### Application roles example

``` json
"ApplicationRoles": [
  {
    "Name": "Appraiser",
    "Description": "User that signs off on the asset price"
  },
  {
    "Name": "Buyer",
    "Description": "User that places an offer on an asset"
  }
]
```
## Identifiers

Identifiers represent a collection of information used to describe workflow properties, constructor, and function parameters. 

| Field | Description | Required |
|-------|-------------|:--------:|
| Name | The unique name of the property or parameter. The corresponding smart contract must use the same **Name** for the applicable property or parameter. | Yes |
| DisplayName | Friendly display name for the property or parameter. | Yes |
| Description | Description of the property or parameter. | No |

### Identifiers example

``` json
"Properties": [
  {
    "Name": "State",
    "DisplayName": "State",
    "Description": "Holds the state of the contract",
    "Type": {
      "Name": "state"
    }
  },
  {
    "Name": "Description",
    "DisplayName": "Description",
    "Description": "Describes the asset being sold",
    "Type": {
      "Name": "string"
    }
  }
]
```

## Configuration file example

The following example defines a basic request-response application in which a requestor sends a request and a responder send a response to the request.

``` json
{
  "ApplicationName": "HelloBlockchain",
  "DisplayName": "Hello, Blockchain!",
  "Description": "A simple application to send request and get response",
  "ApplicationRoles": [
    {
      "Name": "Requestor",
      "Description": "A person sending a request."
    },
    {
      "Name": "Responder",
      "Description": "A person responding to a request"
    }
  ],
  "Workflows": [
    {
      "Name": "RequestResponse",
      "DisplayName": "Request Response",
      "Description": "A simple workflow to send a request and receive a response.",
      "Initiators": [ "Requestor" ],
      "StartState": "Request",
      "Properties": [
        {
          "Name": "State",
          "DisplayName": "State",
          "Description": "Holds the state of the contract.",
          "Type": {
            "Name": "state"
          }
        },
        {
          "Name": "Requestor",
          "DisplayName": "Requestor",
          "Description": "A person sending a request.",
          "Type": {
            "Name": "Requestor"
          }
        },
        {
          "Name": "Responder",
          "DisplayName": "Responder",
          "Description": "A person sending a response.",
          "Type": {
            "Name": "Responder"
          }
        },
        {
          "Name": "RequestMessage",
          "DisplayName": "Request Message",
          "Description": "A request message.",
          "Type": {
            "Name": "string"
          }
        },
        {
          "Name": "ResponseMessage",
          "DisplayName": "Response Message",
          "Description": "A response message.",
          "Type": {
            "Name": "string"
          }
        }
      ],
      "Constructor": {
        "Parameters": [
          {
            "Name": "message",
            "Description": "...",
            "DisplayName": "Request Message",
            "Type": {
              "Name": "string"
            }
          }
        ]
      },
      "Functions": [
        {
          "Name": "SendRequest",
          "DisplayName": "Request",
          "Description": "...",
          "Parameters": [
            {
              "Name": "requestMessage",
              "Description": "...",
              "DisplayName": "Request Message",
              "Type": {
                "Name": "string"
              }
            }
          ]
        },
        {
          "Name": "SendResponse",
          "DisplayName": "Response",
          "Description": "...",
          "Parameters": [
            {
              "Name": "responseMessage",
              "Description": "...",
              "DisplayName": "Response Message",
              "Type": {
                "Name": "string"
              }
            }
          ]
        }
      ],
      "States": [
        {
          "Name": "Request",
          "DisplayName": "Request",
          "Description": "...",
          "PercentComplete": 50,
          "Value": 0,
          "Style": "Success",
          "Transitions": [
            {
              "AllowedRoles": ["Responder"],
              "AllowedInstanceRoles": [],
              "Description": "...",
              "Function": "SendResponse",
              "NextStates": [ "Respond" ],
              "DisplayName": "Send Response"
            }
          ]
        },
        {
          "Name": "Respond",
          "DisplayName": "Respond",
          "Description": "...",
          "PercentComplete": 90,
          "Value": 1,
          "Style": "Success",
          "Transitions": [
            {
              "AllowedRoles": [],
              "AllowedInstanceRoles": ["Requestor"],
              "Description": "...",
              "Function": "SendRequest",
              "NextStates": [ "Request" ],
              "DisplayName": "Send Request"
            }
          ]
        }
      ]
    }
  ]
}
```
## Next steps

> [!div class="nextstepaction"]
> [Deploy Azure Blockchain Workbench](blockchain-workbench-deploy.md)


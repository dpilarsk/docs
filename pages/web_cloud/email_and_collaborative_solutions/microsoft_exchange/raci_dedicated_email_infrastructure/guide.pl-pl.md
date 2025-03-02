---
title: Responsibility model for the product Managed Dedicated E-mail Infrastructure (EN)
excerpt: Shared responsibilities between OVHcloud and the customer
updated: 2024-07-10
---

The RACI below details shared responsibilities between OVHcloud and the customer for the product Managed Dedicated E-mail Infrastructure which is called Trusted Exchange offer on OVHcloud.

This shared model can help relieve the customer’s operational burden.

## RACI definition

| Roles |
| --- |
| R: Is in charge of carrying out the process |
| A: Accountable for the successful completion of the process |
| C: Is consulted during the process |
| I: Is informed of the results of the process |

### 1. Before subscription

#### 1.1. Specify service as needed

| **Activity** | **Customer** | **OVHcloud** |
| --- | --- | --- |
| Learn about capabilities and limitations of the service (documentation, T&C) | RA | I |
| Define resiliency level of the product before subscription | RA | I |
| Define and configure a domain name and owning a domain for which an SSL certificate will be delivered before service subscription | RA | I |
| Define the version of Exchange and the dedicated infrastructure in support of the service |  | RA |
| Host the service in EU |  | RA |

### 2. Service availability

#### 2.1. Install service

| **Activity** | **Customer** | **OVHcloud** |
| --- | --- | --- |
| Install, configure and deliver functional bricks of the service | I | RA |
| Provide service licences and SSL certificate | CI | RA |
| Configure SSL certificate following delivery by OVHcloud | A | R |
| Deliver a preconfigured service with the requested number of mailboxes that need additional configuration by the client | I | RA |

#### 2.2. Reversibility model

| **Activity** | **Customer** | **OVHcloud** |
| --- | --- | --- |
| Choose a migration tool to import email accounts from another service | RA |  |

#### 2.3. Customer Information System setup

| **Activity** | **Customer** | **OVHcloud** |
| --- | --- | --- |
| Customize the service following business needs (domain alerts, quota of accounts, number of accounts, aliases management, define password policy, mail flows, etc.) | RA | I |

### 3. Service usage

#### 3.1. Operations

##### **3.1.1. Daily operations**

| **Activity** | **Customer** | **OVHcloud** |
| --- | --- | --- |
| Manage confidentiality and integrity of data hosted on the service | RA |  |
| Manage network accessibility and data availability on the hosted service following OVHcloud SLA  |  | RA |
| Manage security risks on the service  |  | RA |
| Manage risks of the client software being used (Outlook, Thunderbird, etc.) | RA |  |
| Administrate the service infrastructure (servers, OS, antivirus, storage) |  | RA |
| Administrate domain, mailboxes, distribution groups, a send connector, etc. | RA | IR |
| Manage backups on the service |  | RA |
| Perform (as an option) a backup of mailboxes using a third-party software | RA | I |

##### **3.1.2. Access management**

| **Activity** | **Customer** | **OVHcloud** |
| --- | --- | --- |
| Manage access to the OVHcloud Control Panel and to the dedicated infrastructure following the defined access control policy  | RA | I |
| Define and implement security protocols on the service (IMAP, POP, MAPI, EWS, SMTP) and security associated | I | RA |
| Restrict use of security protocols following business needs (IMAP, POP, MAPI, EWS, SMTP) and security associated | A | R |
| Manage physical and logical access to infrastructures for OVHcloud teams |  | RA |

##### **3.1.3. Monitoring**

| **Activity** | **Customer** | **OVHcloud** |
| --- | --- | --- |
| Monitor and manage alerts for the functioning of systems in support of the service | I | RA |
| Manage the quota on mailboxes |  RA |  |
| Manage hardware sizing on the service |   | RA |
| Maintain logs generated by the service  |   | RA |

##### **3.1.4. Connectivity**

| **Activity** | **Customer** | **OVHcloud** |
| --- | --- | --- |
| Manage IP addressing plan and network systems (architecture, protocols, network traffic and access to the service) |  | RA |
| Filter network flows | RA |  |

##### **3.1.5. Management**

| **Activity** | **Customer** | **OVHcloud** |
| --- | --- | --- |
| Provide inventory of the service used | I | RA |
| Maintain the Exchange service in operational and security condition | I | RA |
| Renew SSL certificate  | A | R |

##### **3.1.6. Business continuity**

| **Activity** | **Customer** | **OVHcloud** |
| --- | --- | --- |
| Define and maintain a business continuity and recovery plan following business needs |  | RA |

#### 3.2. Event management

##### **3.2.1. Incidents**

| **Activity** | **Customer** | **OVHcloud** |
| --- | --- | --- |
| Notify, qualify, and manage incidents on the Exchange service  | I | RA |
| Notify incidents on the Exchange service with a ticketing system  | RA | I |
| Realize backup recovery  | I | RA |

##### **3.2.2. Changes**

| **Activity** | **Customer** | **OVHcloud** |
| --- | --- | --- |
| Plan and deploy updates and patches on the service, OS, and software in support of the service |  | RA |
| Replace deficient hardware in support of the service |  | RA |
| Add and delete hardware and software configuration on the Exchange service |  | RA |

### 4. Reverting

#### 4.1. Reversibility model

| **Activity** | **Customer** | **OVHcloud** |
| --- | --- | --- |
| Choose a migration tool to export email accounts from the old infrastructure  | RA |  |
| Choose the format of extraction file according to the software used for the migration | RA |  |

#### 4.2. Data recovery

| **Activity** | **Customer** | **OVHcloud** |
| --- | --- | --- |
| Define and plan reversion operations | RA | I |

### 5. End of service

#### 5.1. Configuration deletion

| **Activity** | **Customer** | **OVHcloud** |
| --- | --- | --- |
| Delete configurations at end of service following contract termination| I | RA |

#### 5.2. Data deletion

| **Activity** | **Customer** | **OVHcloud** |
| --- | --- | --- |
| Destroy client's data | I | RA |

## Go further

[All our Microsoft collaborative solutions documentation](/products/web-cloud-email-collaborative-solutions-microsoft-exchange)

Join our [community of users](/links/community).

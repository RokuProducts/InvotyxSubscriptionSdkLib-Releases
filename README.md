# 📦 Invotyx Subscription SDK for Roku

The **Invotyx Subscription SDK** provides a unified and reusable interface for managing **Roku Channel Store subscriptions** in **SceneGraph-based Roku applications**.

It enables Roku developers to **initialize the Channel Store**, **fetch available subscription products**, **check user purchases**, and **perform subscription orders** using a simple, consistent API—without directly handling Channel Store complexity.

This SDK is distributed as a **Roku ComponentLibrary**, allowing easy drop-in integration across multiple Roku channels.

---

## 🧩 Overview

The Subscription SDK acts as a centralized subscription layer for Roku apps.  
It abstracts subscription-related interactions and exposes a small set of **high-level methods** and **observable fields** that developers can use to build subscription flows quickly and safely.

### What this SDK helps you do:
- Initialize Roku Channel Store once at app start
- Retrieve available subscription products
- Check existing user subscriptions
- Trigger subscription purchase flow
- Observe purchase results asynchronously

---

## 🧠 Core Features

### 1. Channel Store Initialization
- Prepares the application to interact with Roku’s subscription system
- Must be initialized once before any catalog or purchase calls

### 2. Product Catalog Retrieval
- Fetches all subscription products configured for the channel
- Enables dynamic subscription UI rendering

### 3. Purchase & Subscription Status
- Retrieves current and past user purchases
- Helps determine access control and entitlement logic

### 4. Subscription Ordering
- Initiates Roku’s native subscription purchase flow
- Returns a structured response indicating success or failure

### 5. Observable Data Model
- All results are exposed via observable SceneGraph fields
- Supports clean, event-driven UI updates

---

## 📋 Requirements

- A Roku channel with **in-channel products** configured in the Roku Developer Dashboard
- Access to the **Invotyx Subscription SDK package**
- Valid **Channel Store credentials** provided by Invotyx (SDK owner)

---

## 📥 Integration Guide

### Step 1: Add the ComponentLibrary

Include the SDK using `ComponentLibrary` in your Scene XML.

```xml
<ComponentLibrary
    id="SubscriptionLib"
    uri="https://github.com/RokuProducts/InvotyxSubscriptionSdkLib-Releases/releases/download/v1.0.1/InvotyxSubscriptionSdkLib-Releases.pkg"
/>
```

### Step 2: Observe SDK Load Status

In your Scene BrightScript file, initialize the SubscriptionLib SDK and observe the loadStatus field.

#### Usage
```brs
m.SubscriptionLib = m.top.findNode("SubscriptionLib")
m.SubscriptionLib.observeField("loadStatus", "onLoadStatus")
```

When the SDK is ready, create the Subscription node and initialize it.

```brs
sub onLoadStatus()
    if m.SubscriptionLib.loadStatus = "ready"
        m.global.AddField("SubscriptionLib", "node", false)
        m.global.SubscriptionLib = CreateObject("roSGNode", "InvotyxSubscriptionSdkLib:SubscriptionLib")
        m.global.SubscriptionLib.callFunc("InitChannelStore", "YOUR_STORE_TOKEN")
    else
        print "Subscription SDK failed to load"
    end if
end sub
```

## 🔧 Available API Methods

### 1. `InitChannelStore(token as String)`

Initializes the Roku Channel Store interface.

#### Usage
```brightscript
m.global.SubscriptionLib.callFunc("InitChannelStore", "YOUR_STORE_TOKEN")
```
Note: Must be called before any subscription-related operation.

### 2. `GetAllPurchases()`

Fetches all user subscriptions and purchases.

#### Usage
```brightscript
m.global.SubscriptionLib.callFunc("GetAllPurchases")
m.global.SubscriptionLib.observeField("allPurchases", "OnPurchasesLoaded")
```
```brightscript
sub OnPurchasesLoaded()
    purchases = m.global.SubscriptionLib.allPurchases
    for each purchase in purchases
        print purchase
    end for
end sub
```

### 3. `GetAllCatalog()`

Retrieves all available subscription products for the channel.

#### Usage
```brightscript
m.global.SubscriptionLib.callFunc("GetAllCatalog")
m.global.SubscriptionLib.observeField("productsCatalog", "OnCatalogLoaded")
```
```brightscript
sub OnCatalogLoaded()
    products = m.global.SubscriptionLib.productsCatalog
    for each product in products
        print product
    end for
end sub
```

### 4. `DoSubscriptionOrder(productCode as String, productName as String)`

Initiates a subscription purchase flow.

#### Usage
```brightscript
m.global.SubscriptionLib.callFunc("DoSubscriptionOrder", selectedProduct.code, selectedProduct.name)
m.global.SubscriptionLib.ObserveField("purchaseResponse", "OnSubscriptionOrderComplete")
```
```brightscript
sub OnSubscriptionOrderComplete()
    response = m.global.SubscriptionLib.purchaseResponse
    if response.status = 1
        print "Subscription successful"
    else
        print "Subscription failed"
    end if
end sub
```

### 5. `DoSubscriptionRecovery()`

Triggers a **subscription recovery flow** to restore previously purchased subscriptions for the current Roku user.

#### Usage
```brightscript
m.global.SubscriptionLib.callFunc("DoSubscriptionRecovery")
m.global.SubscriptionLib.observeField("recoveryResponse", "OnRecoveryComplete")
```
```brightscript
sub OnRecoveryComplete()
    response = m.global.SubscriptionLib.recoveryResponse
    if response.status = 1
        print "Subscription recovery successful"
    else
        print "Subscription recovery failed"
    end if
end sub
```

response.status possible values:
-  2: Interrupted
-  1: Success
-  0: Network error
- -1: Server error or timeout
- -2: Timeout
- -3: Unknown Error
- -4: invalid request

## 🔄 Typical Subscription Flow

- Load SDK via ComponentLibrary
- Wait for loadStatus = `"ready"`
- Call InitChannelStore
- Check existing subscriptions `(GetAllPurchases)`
- Fetch available products `(GetAllCatalog)`
- Allow user to select a plan
- Call DoSubscriptionOrder
- Observe purchaseResponse and update UI

## 🧠 Best Practices

- Initialize the Channel Store only once per app lifecycle
- Always check purchases on app start
- Re-fetch catalog when entering subscription screens
- Drive UI updates through field observers
- Treat `status = 1` as the only successful purchase state

## 🧾 Licensing

This SDK is developed and maintained by Invotyx Software Company.
All rights reserved.

Redistribution, modification, or commercial usage is subject to the terms defined by Invotyx.

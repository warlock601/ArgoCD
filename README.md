# ArgoCD
ArgoCD for Canary, Blue-Green &amp; Advanced-Traffic routing using Traefik.



## Limitations of Replica-Weighted Traffic Distribution

- How do you achieve a **10% traffic split** if the ReplicaSet has only **2 pods**?
- To achieve fine-grained rollout steps such as **1%, 2%, or 5%**, you are forced to **over-provision infrastructure**.  
  - Example: if only **2 pods** are running, a **5% split is not practically possible**.  
  - To achieve such precision, you may need **10, 20, or more pods**.
- **No header-based routing:** It is not possible to predictably redirect traffic using **custom request headers**.

---

## Traffic-Weighted Canary

**Traffic-Weighted Canary** is an improvement over **Replica-Weighted Canary**.

It enables **precise traffic weighting** using **Argo Rollouts + Gateway API**.

> Earlier, **Ingress** was also used for this purpose, but that approach is now largely deprecated in favor of **Gateway API**.

In this model, the **traffic distribution logic is handled by the Gateway API**, instead of relying on standard Kubernetes Services.

---

## Working Principle

- A **Stable Service** always points to the **stable environment**.
- A **Canary Service** points to the **canary environment**.
- **Gateway API** performs the actual traffic distribution between them.

Here, **Traefik** is used because it supports **Gateway API**.

### How Traefik Handles Traffic

Traefik reads the **HTTPRoute definition**, specifically the configured **traffic weights**, and distributes requests internally according to those values.

This enables:

- Precise percentage-based routing  
- Independent control from replica count  
- Fine-grained rollout steps such as **95:5**, **99:1**, etc.

---

## Traefik as a Gateway API Controller

**Traefik acts as a Gateway API controller.**

To configure Traefik for Gateway API support:

1. Install **Gateway API CRDs**
2. Install **Traefik with Gateway API enabled**
3. Create a **GatewayClass**
4. Create a **Gateway**
5. Create an **HTTPRoute**


---

## Result

Using this approach, you can achieve precise traffic splits such as:

- **95:5**
- **99:1**
- **90:10**
- even **1% incremental rollout steps**

---

## Argo Rollouts + Gateway API Integration

We create the required **Gateway API resources**:

- **Gateway**
- **HTTPRoute**

Then we define an **Argo Rollout** using the `trafficRouting` block to control that HTTPRoute.

### Request Flow

1. Public traffic enters through the **Gateway**
2. A **PathPrefix match** (for example `/`) sends traffic to the **HTTPRoute**
3. The **HTTPRoute** forwards traffic to:
   - Stable Service
   - Canary Service
4. **Argo Rollouts dynamically updates HTTPRoute weights during rollout**

This means:

> Traffic is distributed based on **percentage**, not based on **number of pods**.

---

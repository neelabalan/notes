# Karpenter Scheduling and Provisioning Behavior

## Overview
Karpenter is a dynamic node provisioning controller for Kubernetes (commonly used with EKS). It does not replace the Kubernetes scheduler.

Process:
- The Kubernetes scheduler attempts to schedule pods.
- If pods cannot be placed (Pending), Karpenter provisions new nodes that fit their requirements.

## No Complex Optimization Involved
Karpenter does not use dynamic programming or knapsack-like optimization.

Instead, it uses straightforward heuristics:
- Identify pending pods and their resource requests.
- Filter the allowed instance types that meet those requests.
- Select the smallest or most cost-effective instance that satisfies the constraints.

## Node Sizing Behavior
Karpenter selects from existing cloud instance types; nodes are never custom-sized.

Example:
If a pod requests:
CPU: 2
Memory: 4Gi

Karpenter may use an instance such as:
- t3.medium (2 vCPU / 4 Gi)
- Or another larger instance if required.

The chosen node simply needs to fit the pod; it may be larger than strictly necessary, but Karpenter aims for the smallest instance that works.

## Configuration Responsibility
You configure which instances Karpenter may choose from, including:
- Instance type filters
- Availability zones
- Architectures
- Spot or On-Demand usage
- Labels and taints

Karpenter only provisions nodes from the options you define.

## Relationship to EKS
EKS manages the Kubernetes control plane but does not automatically handle node creation or removal. Karpenter fills this gap by dynamically provisioning and deprovisioning nodes based on actual pod demand. The Kubernetes scheduler still makes the final pod placement decisions.

## Comparison: EKS vs Karpenter

| Feature                         | EKS Auto Mode (managed)                                                                                                                                                     | Karpenter (self-managed)                                                                        |
| ------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------- |
| Node & Add-On Management        | AWS handles worker node lifecycle, AMI updates, add-ons (CNI, CSI, Load Balancer Controller) for you. ([AWS Documentation][1])                                              | You install and operate Karpenter yourself; you handle AMI choices, add-ons, upgrades.          |
| Instance provisioning & scaling | Uses Karpenter-based system under the hood but is fully managed; you get automatic compute provisioning and scaling without configuring Karpenter. ([AWS Documentation][1]) | You directly configure Karpenter’s instance types, zones, etc, and it reacts to pending pods.   |
| Operational overhead            | Lower: much of the infrastructure management burden is shifted to AWS. ([Repost][2])                                                                                        | Higher: you must maintain Karpenter, the instance fleet, policies, etc.                         |
| Customization & flexibility     | More limited: certain constraints (e.g., AMI choice) and less fine-grained control. ([AWS Documentation][1])                                                                | More flexibility: you can define many custom constraints, instance types, rules.                |
| Cost & visibility               | Provides built-in optimization; but you pay for the managed service layer and may have less direct visibility into underlying autoscaler components.          | You manage costs and visibility yourself; you have full transparency into Karpenter’s behavior. |

[1]: https://docs.aws.amazon.com/eks/latest/best-practices/automode.html?utm_source=chatgpt.com "EKS Auto Mode"
[2]: https://repost.aws/articles/ARpmjGWmwWQuiGg3_NOnfLDg/eks-automode-vs-karpenter?utm_source=chatgpt.com "EKS Automode Vs Karpenter"

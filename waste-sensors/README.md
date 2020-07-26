# waste-sensors

This project collects definitions for potential unrealized savings opportunities, also called waste sensors. Developers are encouraged to contribute new sensors and make enhancements to existing ones.

## Usage

Tools implement their own visualization and remediation code and use the *id* key to get the descriptions from GitHub to be displayed in the user-interface. This is to standardize terms across the FinOps domain. The *id* key is not intended to change over time, however tools should periodically check if the *id* key for a specific implementation still exists on GitHub and raise an exception if it doesn't.

## Keys

### id

The *id* value is considered immutable and uniquely identifies a waste sensor. It is normal for a waste sensor to exist on GitHub and not be implemented by a tool. This means the tool does not yet support the waste sensor. If a tool implemented a waste sensor and the *id* value is not on GitHub, then the CI/CD pipeline should raise an exception and the engineer should either add the waste sensor to GitHub or adjust the *id* value in the tool to match GitHub.

### display_name

This is the short name to be displayed by tools. The *display_name* value is derived from the *id* value and adjusted for proper capitalization.

### type

This is the type of sensor. The *type* value is usually the string "waste-sensor". The key was added for extensibility for other types of sensors that do not readily classify as waste e.g. prepayment product in red zone.

### cloud_provider

The *cloud_provider* value is a set of strings e.g. "aws", "gcp", "azure", "vmcloud", "oracle", "ibm", "alibaba", "rackspace" etc.

The value "independent" is used for waste sensors that are not specific to a cloud provider such as Kubernetes, Splunk etc. As with all waste sensors, the savings opportunity must not overlap with other waste sensors to avoid double counting. For example if Kubernetes runs on EC2, Kubernetes underutilization has to be removed from EC2 underutilization by e.g. using tags.

### description

The *description* value is a short help text that can be displayed by tools in a tooltip.

### comment

The *comment* value is a longer description explaining the savings opportunity in a little more detail e.g. cost avoidance or no discount through prepayment products.

## Waste KPIs

Waste key performance indicators (KPIs) are typically displayed in an aggregated context filtered by tags like account, application, business unit, employee, environment, project and so on.

Each waste sensor aggregates a type of unrealized savings opportunities into two types of KPIs:
- Savings Opportunity
- Waste Percentage

When listing multiple waste sensors it is recommended to sort the list by the largest savings opportunity on the top. This gives users the most impactful recommendations first.

### Savings Opportunity

The savings opportunity represents the actual amount of cost reduction not the total cost of the unoptimized workload. The calculation is fair and accurate and shows the total cost in the local currency that can be avoided after applying all discounts, support and other fees, and the benefit and amortization of prepayment products (e.g. Reservations, Committed Usage Discounts, Savings Plans etc.)

#### Example: aws_ec2_util_low

If the total spend on EC2 in an account is $1,000 and $200 of this spend has a low utilization and can be sized at least one size smaller, then the savings opportunity is $100. This is because the $200 can be reduced in size once, which results in a new cost of $100 and a savings of $100. The waste percentage for this example is $100 / $1,000 = 10%. This is because the $100 is the amount currently in an unoptimized state, which is 10% of the total EC2 cost in the account.

### Waste Percentage

The waste percentage represents how much of a workload is in an unoptimized state. Again the actual amount of cost reduction not the total cost of the unoptimized workload is used and all discounts, support and other fees, and the benefit and amortization of prepayment products are applied. The waste percentage is calculated by using the actual amount of cost reduction as the numerator and the total aggregated context like the account, application, business unit etc. as the denominator.

#### Example: aws_ec2_red_family

Let's say we have two m4.xlarge running in an account. There are no discounts, fees, or prepayment products in effect. An m4.xlarge cost $0.20/hr and an m5.xlarge costs $0.192/hr. The total monthly spend on EC2 in the account is 2 * $0.20 * 24 * 30.5 = $292.80. The savings opportunity is 2 * ( $0.20 - 0.192 ) * 24 * 30.5 = $11.712. The waste percentage is $11.712 / $292.80 = 4%.

If we migrate only one m4.xlarge to m5.xlarge we get a new total monthly spend on EC2 in the account of $0.20 * 24 * 30.5 + $0.192 * 24 * 30.5 = $286.944. The new savings opportunity is now ( $0.20 - 0.192 ) * 24 * 30.5 = $5.856. The new waste percentage is now $5.856 / $286.944 = 2.04%.

## Waste Goal

Specific goals can be set across the organization using Waste KPIs. For example a good starting point is to target a Waste Percentage of 5 percent or less for all workloads in the cloud. These goals can be further customized by tags like account, application, business unit etc.

## Waste Tracking

The performance of workloads toward Waste Goals can be tracked over time. For example the monthly Waste Percentage over the last 12 months can be graphed in conjunction with the target Waste Goal. This allows discussion of trending over time rather than being limited to a snapshot in time.

## Waste Exceptions

Occasionally it may be necessary to grant certain workloads an exception from waste sensor tracking. For example Cassandra clusters are not easily auto scaled and a temporary exception for e.g. 12 months may be appropriate until the team can migrate to a more cloud native workload. These exceptions should be tracked separately to provide leadership visibility into what is currently exempt and needs to be optimized at a later time.

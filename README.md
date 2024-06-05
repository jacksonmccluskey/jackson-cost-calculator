# iot-telemetry-pricing-calculator

This project is for creating a UX template to calculate the estimated price of an IOT telemetry order dependent on telemetry network (ex. satellite constellation), activation fee, monthly subscription fee, unit message fee, number of devices, mobile-originated message interval or total number of mobile-terminated messages, and duration of device deployment.

### Network Fees

_Sample Network Fees Below_

| Network Name (networkName) | Activation Fee ($) (activationFee) | Monthly Fee ($) (monthlyFee) | Unit Message Fee ($) (messageFee) |
| -------------------------- | ---------------------------------- | ---------------------------- | --------------------------------- |
| AwesomeSatelliteNetwork    | 50.00                              | 50.00                        | 0.25                              |
| CoolSatelliteNetwork       | 100.00                             | 100.00                       | 0.50                              |
| AmazingSatelliteNetwork    | 200.00                             | 200.00                       | 1.00                              |

```
interface INetwork {
   networkName: string;
   activationFee: number;
   monthlyFee: number;
   messageFee: number;
}

awesomeSatelliteNetwork: INetwork = {
   networkName: 'AwesomeSatelliteNetwork',
   activationFee: 50,
   monthlyFee: 50,
   unitMessageFee: 0.25,
};

coolSatelliteNetwork: INetwork = {
   networkName: 'CoolSatelliteNetwork',
   activationFee: 100,
   monthlyFee: 100,
   unitMessageFee: 0.50,
};

amazingSatelliteNetwork: INetwork = {
   networkName: 'AmazingSatelliteNetwork',
   activationFee: 200,
   monthlyFee: 200,
   unitMessageFee: 1,
};
```

### Device Deployments

_Sample Deployment Below_
| Network Name (networkName) | Message Interval (messageInterval) | Deployment Duration (deploymentDuration) |
| --- | --- | --- |
| AwesomeSatelliteNetwork| { minutes: 1 } | { months: 6 } |
| CoolSatelliteNetwork | { hours: 2 } | { days: 10 } |
| AmazingSatelliteNetwork | { days: 1 } | { years: 1 } |

```
// Time Related Interfaces

interface ITime {
	minutes?: number;
	hours?: number;
	days?: number;
	months?: number;
	years?: number;
}

type DeploymentDuration = Pick<ITime, 'days' | 'months' | 'years'>;
type MessageInterval = Pick<ITime, 'minutes' | 'hours' | 'days'>;

// Semi-Pseudo Code Below For Pricing Calculations...

minutesInOneHour = 60;

minutesInOneDay = 1440;

daysInOneMonth = 31; // Overestimate; TODO: Incorporate Timeline

daysInOneYear = 366; // Overestimate; TODO: Incorporate Timeline

monthsInOneYear = 12;

calculateMonthsOfDeploymentDuration = ({
	days,
	months,
	years,
}) => {
	monthsInDays = days / daysInOneMonth;

	literalMonths = months;

	monthsInYears  = years * monthsInOneYear;

	return monthsInDays + literalMonths + monthsInYears;
};

calculateTotalServiceCost = ({
	unitFee,
	quantity,
}) => {
	totalServiceCost = unitFee * quantity;

	return {
		unitFee,
		quantity,
		totalServiceCost,
	};
};

calculateTotalActivationsCost = ({
	network,
}) => {
	unitFee = network.activationFee;

	quantity = 1;

	return calculateTotalServiceCost({ unitFee, quantity });
};

calculateTotalMonthlySubscriptionsCost = ({
	network,
	deploymentDuration,
}): ITotalServiceCost => {
	unitFee = network.monthlyFee;

	quantity  = calculateMonthsOfDeploymentDuration({
		days: deploymentDuration.days,
		months: deploymentDuration.months,
		years: deploymentDuration.years,
	});

	return calculateTotalServiceCost({ unitFee, quantity });
};

calculateMessageIntervalInMinutes = ({
	minutes,
	hours,
	days,
}) => {
	numberOfMinutesInMinutes = minutes;

	numberOfMinutesInHours = hours * minutesInOneHour;

	numberOfMinutesInDays = days * minutesInOneDay;

	totalMessageIntervalInMinutes =
		numberOfMinutesInMinutes + numberOfMinutesInHours + numberOfMinutesInDays;

	return totalMessageIntervalInMinutes;
};

calculateTransmissionsPerDayFromMessageInterval = ({
	minutes,
	hours,
	days,
}) => {
	messageIntervalInMinutes = calculateMessageIntervalInMinutes({
		minutes,
		hours,
		days,
	});

	transmissionsPerDay = minutesInOneDay / messageIntervalInMinutes;

	return transmissionsPerDay;
};

calculateDaysOfDeploymentDuration = ({
	days,
	months,
	years,
}) => {
	return (
		days + months * daysInOneMonth + years * daysInOneYear
	);
};

calculateTotalTransmissionsFromDuration = ({
	messageInterval,
	deploymentDuration,
}) => {
	transmissionsPerDay = calculateTransmissionsPerDayFromMessageInterval(messageInterval);

	deploymentDurationInDays = calculateDaysOfDeploymentDuration(deploymentDuration);

	totalTransmissions = transmissionsPerDay * deploymentDurationInDays;

	return totalTransmissions;
};

calculateTotalDataServicesCost = ({
	network,
	deploymentDuration,
	messageInterval,
}) => {
	unitFee = network.messageFee;

	quantity = calculateTotalTransmissionsFromDuration({
		deploymentDuration,
		messageInterval,
	});

	return calculateTotalServiceCost({ unitFee, quantity });
};

calculateTotalCost = ({
	totalActivationsCost,
	totalMonthlySubscriptionsCost,
	totalDataServicesCost,
}) => {
	return (
		totalActivationsCost?.totalServiceCost +
		totalMonthlySubscriptionsCost?.totalServiceCost +
		totalDataServicesCost?.totalServiceCost
	);
};
```

### Customer Interface

```
currentNetwork = 'AwesomeSatelliteNetwork';

currentMessageInterval = {
   days: 1
}

currentDeploymentDuration = {
   months: 6,
}
```

### Activations Cost

```
calculateTotalActivationsCost = ({
	network,
}) => {
	unitFee = network.activationFee;

	quantity = 1;

	return calculateTotalServiceCost({ unitFee, quantity });
};
```

### Monthly Subscriptions Cost

```
calculateTotalMonthlySubscriptionsCost = ({
	network,
	deploymentDuration,
}): ITotalServiceCost => {
	unitFee = network.monthlyFee;

	quantity  = calculateMonthsOfDeploymentDuration({
		days: deploymentDuration.days,
		months: deploymentDuration.months,
		years: deploymentDuration.years,
	});

	return calculateTotalServiceCost({ unitFee, quantity });
};
```

### Data Services Costs

```
calculateTotalDataServicesCost = ({
	network,
	deploymentDuration,
	messageInterval,
}) => {
	unitFee = network.messageFee;

	quantity = calculateTotalTransmissionsFromDuration({
		deploymentDuration,
		messageInterval,
	});

	return calculateTotalServiceCost({ unitFee, quantity });
};

```

### Total Cost

```
calculateTotalCost = ({
	totalActivationsCost,
	totalMonthlySubscriptionsCost,
	totalDataServicesCost,
}) => {
	return (
		totalActivationsCost?.totalServiceCost +
		totalMonthlySubscriptionsCost?.totalServiceCost +
		totalDataServicesCost?.totalServiceCost
	);
};
```

### Invoice (I'm No Accountant...) [0.0.2]

**Activations**

| Device        | Network                 | Activated          | Fee ($)    |
| ------------- | ----------------------- | ------------------ | ---------- |
| AwesomeDevice | AwesomeSatelliteNetwork | 1/1/2025           | 50.00      |
| CoolDevice    | CoolSatelliteNetwork    | 1/1/2025           | 100.00     |
| AmazingDevice | AmazingSatelliteNetwork | 1/1/2025           | 200.00     |
| ---           | ---                     | ---                | ---        |
| ---           | ---                     | **Total Cost ($)** | **350.00** |

**Monthly Subscriptions**

| Device        | Network                 | Plan         | Monthly Fee ($) | Number Of Months | Fee ($)     |
| ------------- | ----------------------- | ------------ | --------------- | ---------------- | ----------- |
| AwesomeDevice | AwesomeSatelliteNetwork | Awesome Plan | 50.00           | 6                | 300.00      |
| CoolDevice    | CoolSatelliteNetwork    | Cool Plan    | 100.00          | 6                | 600.00      |
| AmazingDevice | AmazingSatelliteNetwork | Amazing Plan | 200.00          | 6                | 1200.00     |
| ---           | ---                     | ---          | ---             | ---              | ---         |
| ---           | ---                     | ---          | ---             | **Total ($)**    | **2100.00** |

**Data Services**

| Device        | Network                 | Message Transmission Rate | Number Of Months | Total Number Of Messages | Unit Message Price ($) | Fee ($)      |
| ------------- | ----------------------- | ------------------------- | ---------------- | ------------------------ | ---------------------- | ------------ |
| AwesomeDevice | AwesomeSatelliteNetwork | { minutes: 1 }            | 6                | 267840                   | 0.25                   | 66960.00     |
| CoolDevice    | CoolSatelliteNetwork    | { hours: 2 }              | 6                | 2232                     | 0.50                   | 1116.00      |
| AmazingDevice | AmazingSatelliteNetwork | { days: 1 }               | 6                | 186                      | 1.00                   | 186.00       |
| ---           | ---                     | ---                       | ---              | ---                      | **Total ($)**          | **68262.00** |

**Total Cost**

| Service               | Cost ($)     |
| --------------------- | ------------ |
| Activations           | 350.00       |
| Monthly Subscriptions | 2100.00      |
| Data                  | 68262.00     |
| ---                   | ---          |
| **Total Due ($)**     | **70712.00** |

### Potential Improvements

- Refactor Calculation Methods
- Coding Practices
  - Reusable Components
  - Reusable Methods
  - Standardize
    - Spacing
    - Colors
    - Fonts
    - Dimensions
    - Naming
      - Components
      - Methods
- Remove Duplicates Between Network & Devices
  - Add Device-Dependent Costs
- Add Plan
- Device Status
  - New
  - Activated
    - Add Activation Dates
  - Deactivated
- Message Rate
  - Minute
  - Hour
  - Day
  - Set Maximum
  - Set Minimum
- Message Size
  - Minimum Charge (Included Bytes)
  - Price Per Byte
  - Maximum Bytes Per Message
- Deployment Duration
  - Day
  - Month
  - Year
- Bulk Discounts
  - Monthly
  - Unit Message
- Schedule
  - Deployment Dates (Activations & Deactivations)
  - Message Rate Lifecycle
    - MO
    - MT
- Mobile Terminated (MT) Messages $0.25
  - MT Confirmation (MC) Messages $0.00
- Download Quote As PDF
- Send Email

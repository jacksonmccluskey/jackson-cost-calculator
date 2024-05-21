# iot-telemetry-pricing-calculator

This project is for creating a UX template to calculate the estimated price of an IOT telemetry order dependent on device type, telemetry network (ex. satellite constellation), activation fee, monthly device subscription, number of devices, unit message price, mobile-originated (MO) message interval, number of mobile-terminated messages (variable), and duration of device deployment.

### Device Fees

*Sample Device Fees Below*

| Device Type (deviceType)  | Activation Fee ($) (activationFee) | Monthly Fee ($) (monthlyFee)  | Unit Message Fee ($) (unitMessageFee) |
| --- | --- | --- | --- |
| AwesomeDevice                         | 50.00                                  | 50.00                             | 0.25                       |
| CoolDevice                         | 100.00                                  | 100.00                             | 0.50                       |
| AmazingDevice                         | 200.00                                  | 200.00                             | 1.00                       |

```
interface IDeviceFee {
   deviceType: string;
   activationFee: string; // RESOLVE: Duplicate With Network
   monthlyFee: number; // RESOLVE: Duplicate With Network
   unitMessageFee: number; // RESOLVE: Duplicate With Network
}

// Sample Devices Below

const awesomeDeviceFee: IDeviceFee = {
   deviceType: 'AwesomeDevice',
   activationFee: 50,
   monthlyFee: 50,
   unitMessageFee: 0.25,
};

const coolDeviceFee: IDeviceFee = {
   deviceType: 'CoolDevice',
   activationFee: 100,
   monthlyFee: 100,
   unitMessageFee: 0.5
};

const amazingDeviceFee: IDeviceFee = {
   deviceType: 'AwesomeDevice',
   activationFee: 200,
   monthlyFee: 200,
   unitMessageFee: 1
};
```

### Device Deployments

*Sample Deployment Below*
| Device Type (deviceType)  | Message Interval (messageIntervalInMinutes) | Deployment Duration (deploymentDurationInDays)  |
| --- | --- | --- |
| AwesomeDevice                         | 1440                                  | 186                             | 0.25                       |
| CoolDevice                         | 60                                  | 15                             | 0.50                       |
| AmazingDevice                                       | 15                             | 365.25                       |

```
interface IDeviceDeployment {
   deviceType: DeviceType;
   messageIntervalInMinutes: number;
   deploymentDurationInDays: number;
}

interface IDeploymentDuration {
   days?: number;
   months?: number;
   years?: number;
}

interface ITimeByUnit {
   minutes?: number;
   hours?: number;
   days?: number;
   months?: number;
   years?: number;
}

const minutesInOneHour = 60;

const minutesInOneDay = 1440;

const daysInOneMonth = 31; // Overestimate

const daysInOneYear = 366; // Overestimate

const monthsInOneYear = 12;

const calculateMessageIntervalInMinutes = ({ minutes, hours }: ITimeByUnit) => { // TODO: Error Handling | 24 Hours Max. & 10 Minutes Min.
  return (minutes ?? 0) + ((hours ?? 0) * minutesInOneHour);
}

const calculateTransmissionsPerDayFromMessageInterval = (minutes: number) => {
   return minutesInOneDay / minutes;
}

const calculateDaysOfDeploymentDuration = ({ days, months, years }: IDeploymentDuration): number = {
  return (days ?? 0) + ((months ?? 0) * daysInOneMonth) + ((years ?? 0) * daysInOneYear);
}

const calculateMonthsOfDeploymentDuration = ({ days, months, years}: IDeploymentDuration): number = {
   return ((days ?? 0) / daysInOneMonth) + (months ?? 0) + ((years ?? 0) * monthsInOneYear);
}

const calculateTotalTransmissionsFromDuration = (messageInterval: ITimeByUnit, deploymentDuration: IDeploymentDuration) => {
   const messageIntervalInMinutes = calculateMessageIntervalInMinutes(messageInterval);

   const transmissionsPerDay = calculateTransmissionsPerDayFromMessageInterval(messageIntervalInMinutes);

   const deploymentDurationInDays = calculateDaysOfDeploymentDuration(deploymentDuration);

   return transmissionsPerDay * deploymentDurationInDays;
}

const awesomeDeviceDeployment: IDeviceDeployment = {
   deviceType: 'AwesomeDevice',
   messageIntervalInMinutes: 30,
   deploymentDurationInDays: 15
};

const coolDeviceDeployment: IDeviceDeployment = {
   deviceType: 'CoolDevice',
   messageIntervalInMinutes: 60,
   deploymentDurationInDays: 31, // Entered 1 Month
}

const amazingDeviceDeployment: IDeviceDeployment = {
   deviceType: 'CoolDevice',
   messageIntervalInMinutes: 720,
   deploymentDurationInDays: 366, // Entered 1 Year
}
```

### Networks

```
interface INetwork {
   networkName: string; // AwesomeSatelliteNetwork
   activationFee: number; // RESOLVE: Duplicate With Device Fee
   monthlyFee: number; // RESOLVE: Duplicate With Device Fee
   unitMessageFee: number; // RESOLVE: Duplicate With Device Fee
   deviceTypes: string[];
}

const awesomeSatelliteNetwork: INetwork = {
   networkName: 'AwesomeSatelliteNetwork',
   activationFee: 50,
   monthlyFee: 50,
   unitMessageFee: 0.25,
   deviceTypes: ['AwesomeDevice']
};

const coolSatelliteNetwork: INetwork = {
   networkName: 'CoolSatelliteNetwork',
   activationFee: 100,
   monthlyFee: 100,
   unitMessageFee: 0.50,
   deviceTypes: ['CoolDevice']
};

const amazingSatelliteNetwork: INetwork = {
   networkName: 'AmazingSatelliteNetwork',
   activationFee: 200,
   monthlyFee: 200,
   unitMessageFee: 1,
   deviceTypes: ['AmazingDevice']
};
```

### Customer Interface

```
const currentNetwork = 'AwesomeSatelliteNetwork';

const currentDevice = 'AwesomeDevice'; // RESOLVE: Duplicate Values

const currentMessageInterval: ITimeByUnit = {
   hours: 24
}

const currentDeploymentDuration: IDeploymentDuration = {
   months: 6,
}

const currentNumberOfDevices = 5;
```

### Activations Cost

```
const getTotalActivationsCost = (numberOfDevices: number, network: INetwork) => {
   const totalActivationsCost = currentNumberOfDevices * network.activationFee;
};

const currentTotalActivationsCost = getTotalActivationsCost = (currentNumberOfDevices, currentNetwork);
```

### Monthly Subscriptions Cost

```

const getTotalMonthlySubscriptionsCosts = (numberOfDevices: number, network: INetwork, deploymentDuration: IDuration) = {
   const deploymentDurationInMonths = calculateMonthsOfDeploymentDuration(currentDeploymentDuration);

   return numberOfDevices * network.monthlyFee * deploymentDurationInMonths;
}

const currentTotalMonthlySubscriptionsCosts = (currentNumberOfDevices, awesomeSatelliteNetwork, currentDeploymentDuration);
```

### Data Services Costs

```
const getTotalDataServicesCosts = (numberOfDevices: number, network: INetwork, deploymentDuration: IDuration, messageInterval: ITimeByUnit) => {
   const totalTransmissionsFromDuration = calculateTotalTransmissionsFromDuration(messageInterval, deploymentDuration)

   const totalTransmissionsPerDevice = totalTransmissionFromDuration * network.unitMessageFee;

   return numberOfDevices * totalTransmissionsPerDevice;
}

const currentTotalDataServicesCost = getTotalDataServicesCost(currentNumberOfDevices, awesomeSatelliteNetwork, currentDeploymentDuration, currentMessageInterval);
```


### Total Cost

```
const getTotalCost = (totalActivationsCost: number, totalMonthlySubscriptionsCost: number, totalDataServicesCost: number) => {
  return totalActivationsCost + totalMonthlySubscriptionsCosts + totalDataServicesCost;
};

const currentTotalCost = getTotalCost(currentTotalActivationsCost, currentTotalMonthlySubscriptionsCosts, currentTotalDataServicesCost);
```

###  Invoice (I'm No Accountant...) [0.0.1]

**Activations**

| Device | Network | Activated | Fee ($) |
| --- | --- | --- | --- |
| AwesomeDevice1 | AwesomeSatelliteNetwork | 1/1/2025 | 50.00 |
| AwesomeDevice2 | AwesomeSatelliteNetwork | 1/1/2025 | 50.00 |
| AwesomeDevice3 | AwesomeSatelliteNetwork | 1/1/2025 | 50.00 |
| AwesomeDevice4 | AwesomeSatelliteNetwork | 1/1/2025 | 50.00 |
| AwesomeDevice5 | AwesomeSatelliteNetwork | 1/1/2025 | 50.00 |
| --- | --- | --- | --- |
| --- | --- | **Total Cost ($)** | **250.00** |

**Monthly Subscriptions**

| Device | Network | Plan | Monthly Fee ($) | Number Of Months | Fee ($) |
| --- | --- | --- | --- | --- | --- |
| AwesomeDevice1 | AwesomeSatelliteNetwork | Awesome Plan | 50.00 | 6 | 300.00 |
| AwesomeDevice2 | AwesomeSatelliteNetwork | Awesome Plan | 50.00 | 6 | 300.00 |
| AwesomeDevice3 | AwesomeSatelliteNetwork | Awesome Plan | 50.00 | 6 | 300.00 |
| AwesomeDevice4 | AwesomeSatelliteNetwork | Awesome Plan | 50.00 | 6 | 300.00 |
| AwesomeDevice5 | AwesomeSatelliteNetwork | Awesome Plan | 50.00 | 6 | 300.00 |
| --- | --- | --- | --- | --- | --- |
| --- | --- | --- | --- | **Total ($)** | **1500.00** |

**Data Services**

| Device | Network | Message Transmission Rate | Number Of Months | Total Number Of Messages | Unit Message Price ($) | Fee ($) |
| --- | --- | --- | --- | --- | --- | --- |
| AwesomeDevice1 | AwesomeSatelliteNetwork | 24 Hours | 6 | 180 | 0.25 | 45.00 |
| AwesomeDevice2 | AwesomeSatelliteNetwork | 24 Hours | 6 | 180 | 0.25 | 45.00 |
| AwesomeDevice3 | AwesomeSatelliteNetwork | 24 Hours | 6 | 180 | 0.25 | 45.00 |
| AwesomeDevice4 | AwesomeSatelliteNetwork | 24 Hours | 6 | 180 | 0.25 | 45.00 |
| AwesomeDevice5 | AwesomeSatelliteNetwork | 24 Hours | 6 | 180 | 0.25 | 45.00 |
| --- | --- | --- | --- | --- | **Total ($)** | **225.00** |

**Total Cost**

| Service | Cost ($) |
| --- | --- |
| Activations | 250.00 |
| Monthly Subscriptions | 1500.00 |
| Messages | 225.00 |
| --- | --- |
| **Total Due** | **1975.00**|

### Potential Improvements

- Refactor Calculation Methods (More General)
  - `calculateTransmissionsPerDayFromMessageInterval`
  - `calculateTotalTransmissionsFromDuration`
  - `calculateDaysOfDeploymentDuration`
  - `getDurationInMonths`
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
  - Month
  - Year
- Deployment Duration
  - Day
  - Month
  - Year
- Bulk Discounts
  - Monthly
  - Unit Message
- Schedule
  - Deployment Dates (Activations & Deactivations)
  - Changing Message Rates
    - MO
    - MT
- Mobile Terminated (MT) Messages (MT Confirmation Messages Included)

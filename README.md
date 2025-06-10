# Lara Payment Plan Node.js SDK

This is the Lara Payment Plan SDK, the core of the Lara Credit Proposal System.

## About

This SDK, like other Lara Payment Plan SDKs, is a wrapper around the Lara Payment Plan Rust library.
Currently, the library is available for Linux and Windows. MacOS support is planned for the future.

## Installation
To install the Lara Payment Plan Node.js SDK, you can use npm:

```bash
npm install @parcelado_lara/payment-plan-node
```

The SDK needs to download the compiled Rust library, so you may need to allow the package manager to download it.

## Usage
To use the Lara Payment Plan Node.js SDK, you can import it into your project as follows:

### calculatePlan
Calculates a number of payment plans without down payment given a set of `PaymentPlanParams`.
Returns a `PaymentPlanResponse[]` with the payment plan details, this will a plan for 1 installment up to the number of installments requested,so that you can choose the one that best fits your needs, if the `installmentAmount` of the plan is less than the `minInstallmentAmount` it will not be included in the response.

```javascript
import { calculatePlan } from '@parcelado_lara/payment-plan-node';
const params = {
    maxTotalAmount: Number.MAX_VALUE,
    minInstallmentAmount: 0.0,
    requestedAmount: 8800.0,
    firstPaymentDate: new Date('2022-04-18'),
    disbursementDate: new Date('2022-03-18'),
    installments: 24,
    debitServicePercentage: 0,
    mdr: 0.05,
    tacPercentage: 0.0,
    iofOverall: 0.0038,
    iofPercentage: 0.000082,
    interestRate: 0.0235,
    disbursementOnlyOnBusinessDays: false,
}
const result = calculatePlan(params);
```

Parameters:
- `maxTotalAmount`: Maximum total amount for the plan.
- `minInstallmentAmount`: Minimum installment amount.
- `requestedAmount`: The amount requested for the plan.
- `firstPaymentDate`: The date of the first payment.
- `disbursementDate`: The date of the disbursement.
- `installments`: The number of installments.
- `debitServicePercentage`: Percentage for debit service.
- `mdr`: Merchant Discount Rate.
- `tacPercentage`: Total Amount Charged percentage.
- `iofOverall`: Overall IOF (Tax on Financial Operations).
- `iofPercentage`: IOF percentage.
- `interestRate`: Interest rate for the plan.
- `disbursementOnlyOnBusinessDays`: Whether disbursement is only on business days.

Errors:
- If `requestedAmount` is less than or equal to 0, an error will be thrown.
- If `installments` is less than or equal to 0, an error will be thrown.
- If the any of the dates are invalid
- If any of the tax parameters are too unreasonable for a `Excel XIRR` calculation

### calculateDownPaymentPlan
Calculates a number of payment plans with down payment given a set of `DownPaymentPlanParams`.
Returns a `PaymentPlanResponse[]` with the payment plan details, this will a plan for 1 installment up to the number of installments requested, so that you can choose the one that best fits your needs, if the `installmentAmount` of the plan is less than the `minInstallmentAmount` it will not be included in the response.

```javascript
import { calculateDownPaymentPlan } from '@parcelado_lara/payment-plan-node';
const params = {
    maxTotalAmount: Number.MAX_VALUE,
    minInstallmentAmount: 0.0,
    requestedAmount: 8800.0,
    firstPaymentDate: new Date('2022-04-18'),
    disbursementDate: new Date('2022-03-18'),
    installments: 24,
    downPayment: 1000.0,
    debitServicePercentage: 0,
    mdr: 0.05,
    tacPercentage: 0.0,
    iofOverall: 0.0038,
    iofPercentage: 0.000082,
    interestRate: 0.0235,
    disbursementOnlyOnBusinessDays: false,
}

const downPaymentPlanParams = {
    params,
    requestedAmount: downPayment,
    minInstallmentAmount,
    installments,
    firstPaymentDate: new Date('2022-06-20'),
}

const result = calculateDownPaymentPlan(downPaymentPlanParams);
```
Parameters:
- `requestedAmount`: The amount requested for the down payment.
- `minInstallmentAmount`: Minimum installment for the down payment plan.
- `installments`: The number of installments for the down payment plan.
- `firstPaymentDate`: The date of the first payment for the down payment plan.
- `params`: A `PaymentPlanParams` object containing the parameters for the payment plan(see above).

Errors:
- If `requestedAmount` is less than or equal to 0, an error will be thrown.
- If `installments` is less than or equal to 0, an error will be thrown.
- If the any of the dates are invalid
- If any of the tax parameters are too unreasonable for a `Excel XIRR` calculation

### disbursementDateRange
Calculates a start and end date for a disbursement period based on a `baseDate` and a `numberOfDays`.

```javascript
import { disbursementDateRange } from '@parcelado_lara/payment-plan-node';
const baseDate = new Date('2078-02-12');
const days = 5;
const result = disbursementDateRange(baseDate, days);
console.log(result); // [2078-02-16T03:00:00.000Z, 2078-02-22T03:00:00.000Z ]
/*
  2078-02-12 = Saturday(invalid)
  2078-02-13 = Sunday(invalid)
  2078-02-14 = Bank holiday(invalid)
  2078-02-15 = Bank holiday(invalid)
  2078-02-16 = Wednesday(valid) 1
  2078-02-17 = Thursday(valid) 2
  2078-02-18 = Friday(valid) 3
  2078-02-19 = Saturday(invalid)
  2078-02-20 = Sunday(invalid)
  2078-02-21 = Tuesday(valid) 4
  2078-02-22 = Wednesday(valid) 5


  So the disbursement period is from 2078-02-16 to 2078-02-22.
*/
```
Parameters:
- `baseDate`: The base date from which to calculate the disbursement period.
- `numberOfDays`: The number of days to calculate the disbursement period.
- Returns an array where `[0]` is the start date and `[1]` is the end date of the disbursement period.
- Errors:
  - If `baseDate` is not a valid date, an error will be thrown.

### getNonBusinessDaysBetween
Calculates the non-business(weekends and bank holidays) days between two dates, and returns an array of non-business days.

```javascript
import { getNonBusinessDaysBetween } from '@parcelado_lara/payment-plan-node';
const startDate = new Date('2078-11-12');
const endDate = new Date('2078-11-22');
const result = getNonBusinessDaysBetween(startDate, endDate);
console.log(result); // [2078-11-12T03:00:00.000Z, 2078-11-13T03:00:00.000Z, 2078-11-15T03:00:00.000Z, 2078-11-19T03:00:00.000Z, 2078-11-20T03:00:00.000Z]

/*
  2078-11-12 = Saturday(non-business)
  2078-11-13 = Sunday(non-business)
  2078-11-14 = Monday(business)
  2078-11-15 = Tuesday(non-business)
  2078-11-16 = Wednesday(business)
  2078-11-17 = Thursday(business)
  2078-11-18 = Friday(business)
  2078-11-19 = Saturday(non-business)
  2078-11-20 = Sunday(non-business)
  2078-11-21 = Monday(business)
  2078-11-22 = Tuesday(business)
  So the non-business days are 2078-11-12, 2078-11-13, 2078-11-15, 2078-11-19, and 2078-11-20.
*/
```
Parameters:
- `startDate`: The start date from which to calculate the non-business days.
- `endDate`: The end date until which to calculate the non-business days.
- Returns an array of non-business days between the two dates.
- Errors:
  - If `startDate` or `endDate` is not a valid date, an error will be thrown.
  - If `startDate` is after `endDate`, an error will be thrown.

### nextDisbursementDate
Calculates the next disbursement date based on a `baseDate`

```javascript
import { nextDisbursementDate } from '@parcelado_lara/payment-plan-node';
const baseDate = new Date('2078-02-12');
const result = nextDisbursementDate(baseDate);
console.log(result); // 2078-02-16T03:00:00.000Z
/*
  2078-02-12 = Saturday(invalid)
  2078-02-13 = Sunday(invalid)
  2078-02-14 = Bank holiday(invalid)
  2078-02-15 = Bank holiday(invalid)
  2078-02-16 = Wednesday(valid) 1
*/
```
Parameters:
- `baseDate`: The base date from which to calculate the next disbursement date.
- Returns the next valid disbursement date after the `baseDate`.
- Errors:
  - If `baseDate` is not a valid date, an error will be thrown.

Warning:
As of now, if `baseDate=today` `baseDate` will be considered a invalid date, but this can change in the future.

## Contributing

This repository is public, but the Lara Payment Plan library is private.
To build the library, you must have access to the private repository.
The code here is generated from the private repository and then made public.
Pull requests are welcome, but changes will be made in the private repository and then propagated here.

## License
This software is provided free of charge for personal or internal business use only.
Modification, redistribution, sublicensing, or reverse engineering is not permitted.
Copyright (c) 2025 SWEETPAY SOLUCOES FINANCEIRAS LTDA. All rights reserved.
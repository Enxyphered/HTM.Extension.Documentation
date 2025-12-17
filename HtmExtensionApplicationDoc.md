## HTM Extension Application Installation Guide
### V1.1


The hospitality technology extension application latest version implements the following functions.

```cs
public void ChangeMealPlan(object arg0);
public void BeginByRoom(object arg0);
public void UpdateRoom(object arg0);
public void PrintCheck();
public void SearchByCheckName();
public void SearchRvcItems();
public void RestartServiceHost();
public void OcEntPay();
public void MacroFix();

public event OpsPickUpCheckEvent;
public event OpsDscVoidPreviewEvent;
```

### ChangeMealPlan

```ChangeMealPlan(object arg0)``` has one argument. The argument must be the mealplan name. For example FB, HB, BB, OC or ENT.

The objective of this function is to make sure that you can manually change the bill's mealplan (Order Type and Main Level).

### OpsPickUpCheckEvent

```OpsPickUpCheckEvent``` is an event that is triggered when the bill is opened. When this action is taken the extension application does a few things to ensure some behaviours.

> Check Name Standard: ```$"{room}-{mealPlan}/{guestName}"```

> Based on check name it updates the mealplan using the ```ChangeMealPlan(object arg0)``` function automatically.
It checks for the value between ``'`` and ``/`` to extract the name of the mealplan to do this action.


### BeginByRoom

```BeginByRoom(object arg0)``` function takes a single argument as a prefix of the room. 

Room number will be padded to the left with zeros based on the config.
``PadRoomNumberLeftWithZeros``

For example in Kuramathi they have something called PX rooms that are used after the guests have checked out however the guests have not departed. They have a seperate button to being by PX room number.

If ``config.WarnOnGuestDeparture`` is enabled then user will get a warning that the guest will depart today.

> ⚠️ <span style="color:red;">Warning:</span> If you do not want any prefix before the room number then you should always provide a space character as an argument as no argument is not valid use of this function!

### UpdateRoom

```UpdateRoom(object arg0)``` function takes a single argument as a prefix of the room.

For example in Kuramathi they have something called PX rooms that are used after the guests have checked out however the guests have not departed. They have a seperate button to Update PX Room.

If ``config.WarnOnGuestDeparture`` is enabled then user will get a warning that the guest will depart today.

> ⚠️ <span style="color:red;">Warning:</span> If you do not want any prefix before the room number then you should always provide a space character as an argument as no argument is not valid use of this function!

### PrintCheck

``PrintCheck()`` function takes no arguments as input and it's primary purpose is solve 2 problems.

#### Problem 1: Forgot to give discount when the RVC is an all inclusive RVC

Some customers have the restriction on how many times the bill can be printed without authorization. This function helps to mitigate the issue of a line staff mistakenly printing the bill without anu discount added to the bill. If this happens frequently enough we can use the config option:
> ``ForceDiscountRvcObjectNumbers`` which is a comma seperated list of RVC where this function will be enabled. Any other RVC's will ignore this function and print the bill without asking for confirmation even if there is no discount on the bill.

#### Problem 2: Servicecharge is not present on the bill

The function will detect service charge should be present if all the following conditions are met.

> Total Due > 0<br/>
> Service Charge <= 0<br/>
> RVC Tender Parameters has Automatic Service Charge Selected<br/>
> Bill contains an item that has menu item class option 'Add to auto service charge itemizer' enabled

If all these conditions are met, then it will give the user a warning before the bill is printed.

### SearchByCheckName
``SearchByCheckName()`` takes no argument as a parameter and will search bills by the bill name. This allows the users to pickup the bills by room number instead of table number.


### SearchRvcItems
``SearchRvcItems()`` takes no argument as a parameter and will search for menu items limited by the following parameters:
> RvcObjNumber:
If the revenue center object number is greater than 99 then the object number selected will be the first 2 digits of the rvc number alone.<br/>
> RvcRangeStart = RvcObjNumber * config.RvcRangeMultiplier<br/>
> RvcRangeEnd = (RvcObjNumber + 1) * config.RvcRangeMultiplier - 1<br/>
> PropertyStart = config.PropertyRangeStart<br/>
> PropertyEnd = config.PropertyRangeEnd<br/>

The items filtered by this function will be between ``RvcRangeStart`` and ``RvcRangeEnd`` or ``PropertyStart`` and ``PropertyEnd``.

This function may not work for proeprties not following our standard for configuration.

### RestartServiceHost
``RestartServiceHost()`` takes no argument as a parameter and will restart the ServiceHost. Before it allows you to restart the ServiceHost it will ask the user to input their sign in ID.

> If the user exists<br/>
> If the user has a role<br/>
> If user role is within the comma seperated list ``config.RolesAllowedToRestartServiceHost``<br/>

### OcEntPay
``OcEntPay()`` takes no argument as a parameter and will do the following using the OCENT api at url ``config.OcEntUrl``:

> Send a get request to ``getallaccounts`` endpoint of the OCENT API.
Retreieves a list of accounts you can post to and show the user to select from this list.

> Post the amount to the API then apply an equivilant amount of discount to the bill using disocunt ``config.OcEntDiscountObjectNumber``

### event OpsDscVoidPreviewEvent

The extension application will detect if OCENT discount is voided from any bill and will send a reverse payment to the OCENT API. If the OCENT API is unresponsive the action will fail.

### MacroFix

This function takes no argument and when added to the end of a macro will ensure it does not encounter the issue found in Simphony Version 19.2 where the macro will be stuck and the user will no longer be able to interact with the UI. I believe this issue was resolved after version 19.4 there for can be considered an obsolete/legacy function.
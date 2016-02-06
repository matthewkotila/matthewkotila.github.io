---
layout: post
title: MoneyThisWeek Budget Planner
description: Budget your weekly expenses with your iOS device.

---

MoneyThisWeek is an iOS app for planning your weekly budget. You enter your expenses, and it updates your remaining balance at the start of each week based on your specified budget. It also resets your balance at the beginning of each week and gives easy access to your current balance through a Today Extension.

## The Idea

I'm a college student. I had a part-time job this summer, and the main purpose of it was to make money to spend during the school year. People in my situation have a fixed amount of disposable income. In order to make my earnings last the most efficiently, an idea came to mind. As a fellow debit/credit card user, I have learned that it is much more difficult to keep track of how much I spend when the currency is a piece of plastic that never goes away, as opposed to actual cash, which does. This can be problematic, thus the idea for a budgeting app came to mind.

## Fleshing-Out

The purpose of this app is to provide an easily-accessible and easily modifiable budgeting interface. You input your expenses as you go (after every purchase, once at the end of the day, etc.). The frequency of inputting expenses should be about every day in order to effectively use this app. The app also keeps a record of all transactions entered, and allows for descriptions to be added if you wish to provide some context for an expense.

## Tab One Implementation: Expense

![placeholder](/img/MoneyThisWeekExpenseTab.png "Expense tab")

This app is comprised of three tabs. An Expense tab, a Transactions tab, and a Settings tab. The first tab is the default view shown when the app is launched. This is done on purpose to keep the focus on quick expense input. If you are able to quickly open, submit an expense, and close the app, then its main goal is achieved.

The view itself is simple and contains only a UIButton, UITextField, and two UILabels (one for the current balance, and one for a dollar sign placeholder). From the UITextField, the app updates the current balance stored in NSUserDefaults as well as the UILabel displaying the same stat.

## Tab Two Implementation: Transactions

![placeholder](/img/MoneyThisWeekTransactionsTab.png "Transactions tab")

This tab took the most work to get done correctly. The database for this app is a .plist file stored in the App's documents directory. The second tab is a subclasses UITableViewController, which shows each transaction per cell. Transactions are stored in the plist as an <code>NSArray</code> of <code>NSDictionary</code>s. Each NSDictionary contains three key-value pairs: amount, date, and description. The amount is a float wrapped in an NSNumber. The date is stored as an NSDate created at the time of expense submission, and the description is initially stored as a blank NSString (later to be modified if you set a description). Descriptions can be set by tapping a cell in the UITableView.

![placeholder](/img/TransactionsFile.png "plist storage")

The structure of each transaction is shown above. In order to dynamically populate the <code>UITableViewCell</code>s, the dictionary comes in very handy. Storing data in a plist was the most fitting form of storage for the purpose of an expense log (as opposed to SQLite or NSUserDefaults). Because such storage is only about ⅐ of kilobyte per transaction, it is easily read into memory by the OS. The app has the potential to store thousands of transactions.

## Tab Three Implementation: Settings

![placeholder](/img/MoneyThisWeekSettingsTab.png "Settings tab")

When the app is launched for the first time, you are shown the settings tab. You can enter your weekly budget amount here. Say you're a college student, and you made $1500 over the summer with about 30 weeks in your schoolyear. This leaves about $50 per week to spend—whether it be fun money or only for necessary expenditures. The next option available is choosing the start of your week. This is particularly important for when the app either resets your balance to the specified weekly budget amount or adds the weekly budget amount to the remaining amount at the end of the week depending on which mode is selected.

The two modes are Saver and Spender. Saver works by simply resetting your current balance to the weekly budget amount specified during the first launch. The second mode is Spender mode, which adds the weekly budget amount to the amount remaining at the end of a week.

An example to differentiate between the two modes would be this: You set your weekly budget amount to be $10. The first week you only spent $2. This means you have $8 left over. In Saver mode, the balance at the beginning of the second week would display $10, whereas in Spender mode, the balance would be $18 (the surplus from the previous week rolls over). Although spending over your current week's budget amount is a bad practice, Saver mode still restarts at the weekly budget amount, and Spender mode restarts at the weekly budget amount plus your negative balance from the previous week.

In the last option, you are able to reset all transactions and reset your current week's balance for whatever reason.

## Today Extension/Widget

![placeholder](/img/MoneyThisWeekTodayExtension.png "Today extension")

The Today Extension is comprised of only three elements: a two UILabels and a UIButton. The UILabels are for displaying the current balance quickly updated each time the view is accessed, and the UIButton is an overlay for opening the app when you tap the view. This was my first time working with a Today Extension, and it is quite powerful. Such a small piece of information but located in the right spot can bring so much efficiency to the table as far as balance lookup and app-opening.

## Background Fetch

Another cool piece of technology used for this app was Background Modes. This is found in the Capabilities section of an iOS app. What a background "fetch" allows me to do is have the OS periodically run a piece of code, even if the app is not currently running! This means that the app can check if the new week has been reached, and modify the current balance accordingly—all in the background.

## Beyond

I am hoping to get this in the App Store for others to use and will update this post with links when the time comes.

## The Goods

<img src="/img/MoneyThisWeekAppIcon.png" alt="MoneyThisWeek icon" height="71" width="75">

[Here's](https://github.com/matthewkotila/MoneyThisWeek) the code on GitHub!

<hr>

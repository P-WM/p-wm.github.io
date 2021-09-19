---
title: "Horizon Event"
date: 2021-05-20T00:27:24+01:00
draft: true
---

We live in a world of supposedly ['mutant' algorithms](https://www.bbc.co.uk/news/education-53923279): they mistake [black people for gorillas](https://www.independent.co.uk/life-style/gadgets-and-tech/news/google-photos-tags-black-people-gorillas-puts-pictures-special-folder-10357668.html); they preferentially hire [Jareds who played lacrosse in high school](https://qz.com/1427621/companies-are-on-the-hook-if-their-hiring-algorithms-are-biased/); and recently they even applied grades to [based on prior cohorts' achievement](https://en.wikipedia.org/wiki/Ofqual_exam_results_algorithm). Also, they broke democracy. It used to be that the computer simply said no, but now it sets the limits on your educational attainment, shapes your career path and, in some cases, [even sends you to jail](https://cacm.acm.org/magazines/2021/3/250698-can-the-biases-in-facial-recognition-be-fixed-also-should-they/fulltext).

With this festival of twattery as a backdrop, one could be forgiven for feeling that we've handed over control of increasing parts of our lives to decreasingly accurate machines for the purpose of more effectively marketing dildos and galaxy lamps.


Example of a regression: sometimes when adding a feature (or even fixing an old bug!) one can introduce new bugs affecting old features
Mention Italian cable car accident. Brakes disabled as constantly triggering. Perhaps they were activating every time one strand snapped? Recognise and review errors.

## Remming in

A number of bugs considered by the high court concerned the act or "remming in" money. That is what they called the process by which Post Office branches electronically acknowledged the literal pouches of cash in various denominations they were sent to transact with. An SPM would scan the barcode on the pouch, confirming that they had received the cash and updating Horizon's view of their tills. The amount of cash floating around Post Office branches can be significant and one example often cited concerns a pouch of £8,000. Remming this in multiple times could, therefore, have substantial consequences as Horizon would show an excess £8,000 in the tills every time: £8,000 of money that wasn't there, was never there and could never be there. In the case, in question, this £8,000 pouch was remmed in a surplus three times leading to a 'shortfall' of £24,000: an inappreciable amount for the Post Office but an enormous debt for the individual SPM who was now expected to pay it 'back'.

How could a pouch be remmed in more than once? Two notable bugs were identified: one on the till system in branches and one on Fujitsu's servers. You could consider the former an error with client (a customer's device is acting incorrectly and asking to rem the same cash in multiple times) and the latter an error with a server (Horizon's cloud is allowing the same cash to be remmed in multiple times).

### The cloud
The remming in error with Horizon's cloud was simple but pernicious. The Horizon database had a table with the IDs of all the pouches that had been sent out to branches along with the date and time (or `timestamp`) they were remmed in. If the datetime for a given pouch were empty (or `NULL`), it had not yet been remmed in and its cash could be safely added to the branch's account. Before settling any pouch, the software would check if it had already been remmed in by sending the database its unique ID and asking for its settled timestamp back. If the result was not `NULL` then the pouch must already be remmed in and it must not happen again. This was called the `SettlePouchDeliveryPreCheck`: before we settle the pouch (i.e. update the branch's accounts) we check if it's valid to do so.

Except that the software did not send the pouch's ID to the database: it instead sent its barcode. This is a simple mistake. A programmer was working with several pieces of data relating to a pouch (including one called `pouchId` and one called `pouchBarcode`). When they wrote the code for the `SettlePouchDeliveryPreCheck` they wrote the wrong one and ended up checking the pouch's settled status using a query that would always return `NULL` (indicating it was safe to settle). Both `pouchId` and `pouchBarcode` relate to the same pouch, both uniquely identify it and both were correctly known by the software but the incorrect one was used. 

A simple analogy is perhaps an app to confirm vaccination status. Imagine a system that sends my NHS number to a database and checks the date I had my COVID jab. If a date is returned, then I have some level of protection. Imagine, however, that due to a similar programming error, the software instead sends my National Insurance number. Since there is no record of vaccination for that number (it's completely the wrong format for a start), then no date would be returned. Both my NI number and NHS number are related to me, they both uniquely identify me and this putative system may well know both correctly. However, my NI number is the wrong identifier to use for this query and cannot return the correct result meaning this app would never show me as vaccinated.

This simple mistake (essentially a typo) sent a pregnant woman to jail.

#### Mitigations

How could this have been avoided? One important step is testing. SAY MORE

We often write, for instance, unit tests (that demonstrate whether or not a single piece of code functions correctly), integration tests (that demonstrate with several components work together correctly to perform BLAH BLAH) and end to end (or E2E) tests which create a facsimile of the whole system and attempt to perform certain actions exactly as a user would.

If I were working on the `SettlePouchDeliveryPreCheck`, I would expect to see at the very least one test showing that the check passes for a pouch that has not been settled and one test showing that it fails for one that has. Furthermore, I would like to have one test showing that a pouch can be settled once and one test showing that it cannot be settled more than once. That is, instead of my test directly checking if the `SettlePouchDeliveryPreCheck` gives the correct result, it would instead simply try to settle a pouch more than once and check that it fails.

The devil, however, is in the details.

Note clearly possible gaps in first one. Note also gaps in the second (e.g. difference between setting up state where already settled then settling again vs. a test that settles once and then tries to settle again)

#### Race condition
There is also the possibility of something called a 'race condition'; this is where the order in which events take place is important but cannot be predicted or controlled. In the case of remming in a pouch of money, a race condition occurs as two attempts to rem in a pouch may be initiated (and receive a satisfactory pre check) before any is complete. The order of events would be as follows:

1. Pouch A checks for a timestamp

From the database's perspective (once the vicissitudes of the internet are taken into account), the order of the requests may well be:

1. Request 1 for Pouch A's timestamp
2. Request 2 for Pouch A's timestamp
3. Set Pouch A's timestamp
4. Set Pouch A's timestamp

mention:
- tests
- dangers of mocking
- race condition (update where pouchid = pouchid and timestamp is null)


Initially [39 cases](https://ccrc.gov.uk/ccrc-to-refer-39-post-office-cases-on-abuse-of-process-argument/) referred to court of appeal (citing abuse of process) by CCRC (at the time, the largest single referral in its history), then a [further eight](https://ccrc.gov.uk/the-ccrc-refers-eight-more-post-office-cases-for-appeal-bringing-total-to-47-so-far/). Post Office is now reviewing a further 900 prosecutions!

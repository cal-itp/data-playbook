# Informing customers about fares + payment in trip planners

Transit riders deserve to know as much information as possible when they plan their journey, including fares.  

* Trip planning applications [leverage standardized and published GTFS datasets](https://www.transitwiki.org/TransitWiki/index.php/Category:GTFS-consuming_applications) to transmit information to customers.  
* Fare information, such as what to pay and how, can be described through the [GTFS-Fares v2](http://bit.ly/gtfs-fares) extension (adopted Fall 2021) to the [General Transit Feed Specification](http://gtfs.mobilitydata.org) (GTFS) standard.

![How transit traveller information gets to customers: transit provider to GTFS.zip files to trip planning application on customer mobile phone to customer.](/img/gtfs-to-rider.png)
*How transit traveller information gets to customers (Image: Cal-ITP)*

!!! warning "Fares v2 not yet fully supported"
    Fares v2 is not yet supported by all trip planning applications.  
    Trip planning applications which support portions of Fares v2 include:

    - Transit App

    [More details about which features are adopted supported.](gtfs-fares.md#what-gtfs-files-does-the-fare-information-live-in)

The rest of this Playbook entry contains information on:

1. [Process to code and publish fare data in a GTFS dataset](gtfs-fares.md/#process-overview)
2. [GTFS Fares v2 data standard](/gtfs-fares.md/#gtfs-fares-files)
3. [Coding strategies by use case](gtfs-fares.md/#handling-specific-fares-use-cases)
4. [Example fare data](gtfs-fares.md/#example-fares-v2-datasets)
5. [Additional resources](gtfs-fares.md/#additional-resources)

## Process Overview

| Step                                                                       | Description                                                                                                                            |
| :------------------------------------------------------------------------- | :------------------------------------------------------------------------------------------------------------------------------------- |
| [**Scope + Information Gather**](gtfs-fares.md#scoping)                    | Research online and interview agency staff to clarify and confirm how their fare system works.                                         |
| [**Import from existing CSV**](gtfs-fares.md/#importing-existing-gtfs)     | If you need to express network or zone-based fares or if you are modifying existing Fares v2 files.                                    |
| [**Add or update data**](gtfs-fares.md/#handling-specific-fares-use-cases) | Add data to a fare data template ([some examples](gtfs-fares.md#additional-resources)) or your own tooling based on use-cases          |
| [**Export to CSV**](gtfs-fares.md/#export-from-a-spreadsheet)              |                                                                                                                                        |
| [**Validate**](gtfs-fares.md/#validate-gtfs-fares-v2-data)                 | Make sure it complies with the spec and trip planners can ingest it                                                                    |
| [**QA/QC**](gtfs-fares.md/#validate-gtfs-fares-v2-data)                    | make sure it accurately reflects what you want it to                                                                                   |
| [**Publish**](gtfs-fares.md/#publishing-fares-v2-in-a-gtfs-feed)           | with the GTFS Schedule dataset                                                                                                         |
| Business process update                                                    | Make sure published fares in GTFS remain up-to-date by establishing or renewing the business processes that will make sure it happens. |

## Scoping

### Identify fare types

A typical process for identifying use cases and getting their details includes both:  

* Reviewing information on the transit provider's website, and  
* Contacting the transit provider to fill in gaps and confirm website information validity.  

In some simple cases, it may be easiest to move ahead and code the fares absent confirmation of the transit provider first and to contact them when conducting the QA/QC process to confirm data accuracy.

### Resource planning: GTFS Fares v2 level of effort

| Use Case                                                       | Level of Effort           | Comments                                                                                                                                                                             |
| :------------------------------------------------------------- | :------------------------ | :----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| [Free Fares](gtfs-fares.md/#free-fares)                        | 🌶️                     | Easy like Sunday morning.                                                                                                                                                            |
| Simple flat fares                                              | 🌶️🌶️                | Can often be created in less than an hour.                                                                                                                                           |
| [Payment type](gtfs-fares.md/#stored-value-fare_containerstxt) | 🌶️🌶️🌶️           |                                                                                                                                                                                      |
| [Ticket type](gtfs-fares.md/#fare-products-fare_productstxt)   | 🌶️🌶️🌶️           |                                                                                                                                                                                      |
| [Rider type](gtfs-fares/#groups-of-riders-rider_categoriestxt) | 🌶️🌶️🌶️           |                                                                                                                                                                                      |
| [Route-type fares](tfs-fares.md/#route-based-fares)            | 🌶️🌶️🌶️           | Slightly difficult processes-wise because of need to edit existing GTFS files.                                                                                                       |
| [Zone-based fares](gtfs-fares.md/#zone-or-stop-based-fares)    | 🌶️🌶️🌶️🌶️      | Overlaping fare zones or zones which are single stops can become quite difficult to manage and QA/QC. Slightly difficult processes-wise because of need to edit existing GTFS files. |
| Combination of the above                                       | 🌶️🌶️🌶️🌶️🌶️ | Feeds which combine several facets are more difficult to code and QA/QC.                                                                                                             |

## GTFS Fares files

The [GTFS standard](https://gtfs.mobilitydata.org) describes transit operations through a set of comma-separated value (CSV) files. Each file has a predetermined set of columns, and the data that fills its rows must follow the rules dictated in the standard. GTFS-Fares v2 adds files, and columns to existing files to the base GTFS standard as decribed below.  Some files or fields are optional depending on the system you are trying to describe.  Additionally, Some files or fields, as noted below, are not yet a "fully adopted" part of the GTFS specification and are subject to modification.  

| File                                                                                                                  | Adoption Status                                       | Description                                                                                                                |
| :-------------------------------------------------------------------------------------------------------------------- | :---------------------------------------------------- | :------------------------------------------------------------------------------------------------------------------------- |
| [fare_leg_rules.txt](gtfs-fares.md#describe-core-fare-information-in-fare_leg_rulestxt)                               | Adopted. Supported in some trip planners.             | Required, even if only to explicitly state that fares are free.                                                            |
| [rider_categories.txt](gtfs-fares.md#describe-groups-of-riders-eligible-for-specialized-fares-in-rider_categoriestxt) | Unadopted. Draft subject to change.                   | Required if not all riders are eligible for the same fares (e.g. senior discounts).                                        |
| [fare_containers.txt](gtfs-fares.md#describe-stored-value-like-bank-and-transit-cards-in-fare_containerstxt)          | Unadopted. Draft subject to change.                   | Required to describe different ways for riders to use stored value other than cash (e.g. bank cards, transit cards, etc.). |
| [fare_products.txt](gtfs-fares.md#describe-products-like-multi-ride-passes-in-fare_productstxt)                       | Unadopted. Draft subject to change.                   | Required to describe products other than a single ride fare (e.g. discounted passes, multi-ride tickets, etc.)             |
| [fare_capping.txt](gtfs-fares.md#describe-fare-cut-offs-in-fare_cappingtxt)                                           | Unadopted. Draft subject to change.                   | Required to describe a rule where riders don’t pay more than a specified maximum amount over a given period of time.       |
| [fare_transfer_rules.txt](gtfs-fares.md#describe-transfer-rules-using-fare_transfer_rulestxt)                         | Adopted. Supported in some trip planners.             | Relationship between fare leg groups if transferring.                                                                      |
| [references.txt](https://bit.ly/gtfs-references)                                                                      | Under active development. No current implementations. | Transfer fare agreements for transit not represented within the subject GTFS dataset                                       |
| [routes.txt](gtfs-fares.md#differentiate-fares-across-routes-in-routestxt)                                            | Adopted. Supported in some trip planners.             | Existing file.<br>Modifications required to encode route-based fares such as premium express buses.                        |
| [stops.txt](gtfs-fares.md#)                                                                                           | Adopted. Supported in some trip planners.             | Existing file.<br>Modifications required to encode stop- or zone-based fares.                                              |

The following sections describe each of these files in more detail.

### Groups of riders: `rider_categories.txt`

Each row describes a different kind of rider that can be eligible for a discount or different fare.  This file is required if the provider offers persona-based discounts, and should not be published if these are not part of the provider’s fare policy.

The most common categories are seniors, persons with disabilities, and youth. Some transit providers also include veterans and Medicare cardholders, which are each their own category. When providers offer free riders to children under a certain age or height, that can be coded as well.  

??? info "Fields in rider_categories.txt"
    | Field               | Requirements | Description                                                                                                                                                                                                                |
    | :------------------ | :----------- | :------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
    | rider_category_id   | Required     | A short ID. Riders won’t see this field. Examples include: “senior” and “disabled”.                                                                                                                                        |
    | rider_category_name | Required     | A human-readable description of the category that riders will see which is consistent with how they’re described on the transit provider’s website. Examples include: “Seniors (age 65+)” and “Persons with Disabilities.” |
    | min_age             |              | Riders must be at least this old to receive the discount. Seniors often have a minimum age around 65.                                                                                                                      |
    | max_age             |              | Riders must be younger than this to receive the discount. Youth often have a maximum age of 17.                                                                                                                            |
    | eligibility_URL     |              | If the transit provider’s website has a page that explains more about this rider category, such as an application, link to it here.                                                                                        |

??? example "rider_categories.txt for Eastern Sierra Transit Authority"
    | rider_category_id | rider_category_name                             | min_age | max_age | eligibility_url                                          |
    | :---------------- | :---------------------------------------------- | :------ | :------ | :------------------------------------------------------- |
    | senior            | Seniors Age 60 and Over                         | 60      |         | https://www.estransit.com/fares-and-reservations-policy/ |
    | disabled          | Persons with Disabilities                       |         |         | https://www.estransit.com/fares-and-reservations-policy/ |
    | youth             | Youth Ages 5-15                                 | 5       | 15      | https://www.estransit.com/fares-and-reservations-policy/ |
    | children          | Children Age 4 and Under with Fare Paying Adult |         | 4       | https://www.estransit.com/fares-and-reservations-policy/ |

### Stored value: `fare_containers.txt`

Each row describes a different way for riders to store value. This file is required if the provider offers fare containers, and should not be published if these are not part of the provider’s fare policy.

Clipper and TAP are both stored value cards, and each would have its own row. Cash is always assumed to be an option, so it doesn’t need to be explicitly coded as a fare container. Caltrans considers contactless bank cards to be fare containers.

??? info "Fields in fare_containers.txt"
    | Field                    | Requirements                                                                                  | Description                                                                                                                                                                                                                                                                 |
    | :----------------------- | :-------------------------------------------------------------------------------------------- | :-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
    | fare_containers_id       | Required                                                                                      | A short ID. Riders won’t see this field. Examples include: “Clipper”, “TAP”, and “Bank”                                                                                                                                                                                     |
    | fare_container_name      | Required                                                                                      | A human-readable description of the container that riders will see. Examples include: “Clipper Card”, “TAP Card”, and “Contactless Mastercard/Visa”.                                                                                                                        |
    | minimum_initial_purchase | Required to describe what the minimum initial value is.                                       | If riders must buy the container with some initial purchase, this field is the cost of that stored value. Don’t include the “$”. <br> Example: If a new card requires a $5 initial purchase, and this will go towards $10 of rides with the card, this field should be “5”. |
    | amount                   | Required to describe the price of the container itself.                                       | Describes the price of the fare container itself. Example: A new card costs $3, and the new card has an initial balance of $0. This field should be “3”.                                                                                                                    |
    | currency                 | Required if `minimum_initial_purchase` or `amount` are non blank. Otherwise, should be blank. | `USD` in the United States.                                                                                                                                                                                                                                                 |
    | rider_category           | Required if certain riders are eligible for this container.                                   | List the ID of that rider_category that is eligible here. <br>Example: Adults pay $5 for a new card, but seniors pay $3. This would be two different rows. One would be $5 with no rider category, and the other would be 3 with the senior rider category.                 |

??? tip "Multiple Fare Containers"  
    All fare products (e.g. a day pass) must be entered twice in fare_products.txt if there are two possible fare containers (or lack thereof) for the products.  

    The most common example of two possible fare containers is a paper pass and a digital pass.

    In this case, the digital day pass (e.g. a Connect card) is defined in fare_containers.txt but the paper day pass is not.  Both options have their own separate entry in fare_products.txt. This tells the consuming application that the same day pass can be bought and used in two unique ways.

    Failing to enter in all fare products twice conveys that a rider can ONLY buy either a paper day pass or a digital day pass on a reloadable Connect card (depending on whether fare_container is defined or not). In reality, riders have both options available to them so the fare products must be entered twice.

??? example "fare_containers.txt for AC Transit (San Francisco Bay Area)"
    | fare_container_id | fare_container_name | minimum_initial_purchase | amount | currency | rider_category_id |
    | :---------------- | :------------------ | :----------------------- | :----- | :------- | :---------------- |
    | clipper           | clipper             |                          | 3      | USD      |                   |

??? example "fare_containers.txt for Monterey Salinas Transit"
    | fare_container_id | fare_container_name         | minimum_initial_purchase | amount | currency | rider_category_id |
    | :---------------- | :-------------------------- | :----------------------- | :----- | :------- | :---------------- |
    | bank              | Contactless Mastercard/Visa |                          |        |          |                   |

Additional information:

* [Detailed instructions for fare products use case.](gtfs-fares.md#passes-transit-cards)  

### Fare products: `fare_products.txt`

Most transit providers offer fare products: discounted passes or multi-ride ticket books.  

fare_products.txt documents:

* how the products are priced,  
* who is eligible,  
* when they’re valid, and  
* how the provider describes them.  

This file is required if the provider offers fare products, and should not be published if these are not part of the provider’s fare policy.

??? info "Fields in fare_products.txt"
    | Field             | Requirements                                                                                                                           | Description                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                   |
    | :---------------- | :------------------------------------------------------------------------------------------------------------------------------------- | :-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
    | fare_product_id   | Required                                                                                                                               | A short ID. Riders won’t see this field. Examples include: “day” and “senior monthly”                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                         |
    | fare_product_name | Required                                                                                                                               | A human-readable description of the product that riders will see, e.g. “Day Pass” and “Monthly Pass (Seniors)”.<br>Specify if the product is for a special rider category. If it’s for adults (no named rider category), use the provider’s own language to describe this. For example, “general”, “adult”, or “regular”. <br> One `fare_product_i`d can be tied to multiple `fare_product_names`. For example: A fare_product_id “discount” could be linked to rows with fare_product_names “senior” and “disabled”, if those products work the same way for both of those groups of riders. |
    | rider_category_id | Required if this pass is only available to certain riders.                                                                             | Create a row for each rider category. Rider categories can share a fare_product_id, just like fare_product_name above. <br> Example: Both seniors and persons with disabilities can buy a discounted monthly pass.<ul><li>`fare_product_id`: "discount monthly"; `rider_category`: "senior"</li><li>`fare_product_id`: "discount monthly"; `rider_category`: "disabled"</li></ul>                                                                                                                                                                                                             |
    | fare_container_id |                                                                                                                                        | If the rider must have a specific fare container to use the pass, that ID is required here. Fare containers can share a fare_product_id, too.                                                                                                                                                                                                                                                                                                                                                                                                                                                 |
    | bundle_amount     |                                                                                                                                        | If the product is a pass good for a certain number of rides, put that number here. For example, for a ten-ride pass, this field is “10”.                                                                                                                                                                                                                                                                                                                                                                                                                                                      |
    | duration_start    | Required if the product is valid for some set period of time, and that time is based on defined periods of time (like calendar months) | Options include:<ul><li>0: If the product needs to wait until some date to become active, such as the first of the month</li><li>1:If the product can be used immediately after purchase.</li></ul>                                                                                                                                                                                                                                                                                                                                                                                           |
    | duration_amount   | Required if the product is valid for some set period of time.                                                                          | This field describes the number of time units that the product is valid. For example, a pass that’s valid for 31 days would be “31” or  pass that’s valid for 1 month would be “1”.                                                                                                                                                                                                                                                                                                                                                                                                           |
    | duration_unit     |                                                                                                                                        | If the product is valid for some set period of time, this field is required. This field describes the time unit that the product is valid. The options are:<ul><li>0: Seconds</li><li>1: Minutes</li><li>2: Hours</li><li>3: Days, e.g. a pass that is valid for 1 day</li><li>4: Weeks</li><li>5: Months, e.g. a pass that is valid for 1 month </li><li>6: Years</li></ul>                                                                                                                                                                                                                  |
    | duration_type     | Required if the product is valid for some set period of time.                                                                          | This field describes the window for which a fare product is valid. Options are: <ul><li>1: If the window is based on the calendar. e.g. A day pass is activated at 10 AM. The pass is valid until midnight</li><li>2: If the window is based on the time of activation. e.g. A day pass is activated at 10 AM. The pass is valid until 9:59 AM the next day.                                                                                                                                                                                                                                  |  
    | amount            | Required                                                                                                                               | The cost of the pass. Don’t include a “$”.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                    |
    | currency          | Required                                                                                                                               | `USD` in the United States.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                   |

=== "One set of fares products"
    If there is only one set of fare products available, those products are assumed to be from the agency of the dataset we are working in.  

    !!! example "One set of fare products: Placer County Day Passes"

        Day passes in the Placer County gtfs dataset are assumed to be Placer County's fare products.

        In these cases, name and ID do not need to be specified, although it is helpful if they are. ('Day Pass Senior' is sufficient and does not need to be further specified as 'Placer County Day Pass Senior').

=== "Two or more sets of fare products"
    If there are two or more sets of fare products available, name and ID must be detailed and specify which agency a rider is buying a product from.  

    !!! example "Two sets of fare prodcuts: Placer County and SacRT Passes"

        'Placer County Day Pass Senior' and  'Sacramento Regional Transit Day Pass Senior'.  

??? tip "Scrip Tickets: Coupons, Paper Stored Value"

    Cash can sometimes be exchanged for a paper product that can then be used to pay your fare. These paper products SHOULD be included if cash is exchanged for a paper bus ticket (Ex: $1 for 1 ticket, $10 for 10 tickets). These are often called bus booklets and bundle_amount can be defined.

    These paper products should NOT be included if cash is merely exchanged for paper slips with dollar amounts on them. The difference is that these slips are not bus tickets, they merely allow the rider to use them to pay their fare. This is important if these paper slips cannot cover the cost of a fare, if it is not a 1:1 ratio ($1 - 1 ticket - 1 fare), and if there are odd change situations that arise due to their use (Ex: A rider can pay $10 for 10, $1 scrip tickets but the bus fare is $1.75). In these cases, bundle_amount cannot be defined.

    It is recognized that such scrip tickets and coupons are important for low-income riders who do not want to carry around large amounts of cash but they will not be included for the time being due to their complicated nature.

??? example "fare_product.txt for Big Blue Bus"

    | fare_product_id | fare_product_name       | rider_category_id | fare_container | bundle_amount | duration_amount | duration_unit | duration_type | amount | currency |
    | :-------------- | :---------------------- | :---------------- | :------------- | :------------ | :-------------- | :------------ | :------------ | :----- | :------- |
    | day             | Day Pass (Regular)      |                   | TAP            |               | 1               | 3             | 1             | 4      | USD      |
    | day             | Day Pass (Senior)       | senior            | TAP            |               | 1               | 3             | 1             | 1.5    | USD      |
    | 13-ride         | 13 Rides Pass (Regular) |                   | TAP            | 13            |                 |               |               | 14     | USD      |
    | 13-ride         | 13 Rides Pass (Senior)  | senior            | TAP            | 13            |                 |               |               | 6      | USD      |
    | annual          | Annual Pass (Regular)   |                   | TAP            |               |                 | 6             | 2             | 500    | USD      |

Additional information:

* [Detailed instructions for fare products use case.](gtfs-fares.md#passes-transit-cards)  

### Fare cut-offs: `fare_capping.txt`

Some transit providers limit the amount a rider can pay for transit in a day. For example, if each ride costs $2, and the daily “cap” is $5, the rider would pay $2 for their first two rides in a day, then $1 for the third, and then ride the rest of the day for free.  

GTFS can describe these rules with the optional `fare_capping.txt` file. If fare capping is not part of the provider’s fare rules, this file should not be published.

??? info "Fields in fare_capping.txt"
    | <div style="width:150px">Field</div> | Requirements | Description                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                  |
    | :----------------------------------- | :----------- | :----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
    | fare_product_id                      | Required     | The capped fare must be connected to a fare product. For example, the daily cap may be the same as a daily pass. In this case, the rider effectively “earns” a daily pass once they’ve paid the equivalent amount in individual fares.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                       |
    | eligible_cap_id                      | Required     | A short name to describe this capping rule. Will not be visible to the rider.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                |
    | fare_container_id                    | Required     | The capped fare must also be connected to a container. Cash fares cannot count towards capping.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                              |
    | duration_amount                      | Required     | "The amount of time units within the fares are capped. Similar to how time is calculated in `fare_products.txt`                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                              |
    | duration_unit                        | Required     | The unit for which time is capped. The options are: <ul><li>0: Seconds</li><li>1: Minutes</li><li>                                                                                                             2: Hours</li><li>                                                                                                             3: Days</li><li>4: Weeks</li><li>                                                                                                             5: Months                                                                                                              </li><li>6: Years                                                                                                               </li></ul> |
    | duration_type                        |              | Defines if the duration is calendar-based or rolling. Options:<ul><li>1: calendar-based, expires based on an absolute time or date (i.e. midnight, last day of the month, etc.)</li><li>2: rolling, expires based on when it started (i.e. any 24 time period, any 7 day-time period, etc)</li></ul>                                                                                                                                                                                                                                                                                                                                                                                                                                                         |
    | cap_amount                           | Required     | The amount a rider has to pay during the time period before they no longer have to pay. Likely equal to the value of the equivalent pass.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                    |
    | currency                             | Required.    | `USD` in the United states.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                  |

??? example "fare_capping.txt for County Connection"

    A rider pays fares all day Monday. They earn a day pass, which expires at midnight. The count restarts for Tuesday.

    | fare_product_id | eligible_cap_id | rider_category_id | fare_container | duration_amount | duration_unit | duration_type | cap_amount | currency |
    | :-------------- | :-------------- | :---------------- | :------------- | :-------------- | :------------ | :------------ | :--------- | :------- |
    | day             | day_cap         |                   | clipper        | 1               | 3             | 1             | 3.75       | USD      |

### Differentiate fares across routes: `routes.txt`

Some transit agencies charge different amounts for different routes. For example, the local bus might be $1, but the express might be $2. This can be described with GTFS-Fares, but requires making an addition to the existing routes.txt file.

??? info "Relevant fields in routes.txt"

    | <div style="width:150px">Field</div> | Description                                                                                                                                                                   |
    | :----------------------------------- | :---------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
    | route_id                             | Primary key for `routes.txt`                                                                                                                                                  |
    | network_id                           | Referenced in `fare_leg_rules` to tie specific transit routes to specific one-way fares. Can be text based and should describe the network multiple transit routes belong to. |

network_id field is not especially difficult to design, but does require more care to add it to the GTFS feed before publication.

Additional information:

* [Detailed instructions for route-based fares use case.](gtfs-fares.md#route-based-fares)  
* [Full routes.txt documentation.](https://gtfs.mobilitydata.org/spec/gtfs-schedule#h.elenes7l8gmy)

### Fare zones: `stops.txt`

Fares which vary based on origin- or destination- locations must encode a zone_id into relevant boarding and alighting stops as well as any stops that must be traversed to activate certain fares using [fare_leg_rules.contains_area_id](gtfs-fares.md/#fare-rules-fare_leg_rulestxt).

??? info "Relevant fields in stops.txt"

    | <div style="width:150px">Field</div> | Description                                                                                                                                                                                                                                            |
    | :----------------------------------- | :----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
    | stop_id                              | Primary key for stops.txt                                                                                                                                                                                                                              |
    | zone_id                              | Referenced in [fare_leg_rules.from_area_id](gtfs-fares.md/#fare-rules-fare_leg_rulestxt), [fare_leg_rules.to_area_id](gtfs-fares.md/#fare-rules-fare_leg_rulestxt), or [fare_leg_rules.contains_area_id](gtfs-fares.md/#fare-rules-fare_leg_rulestxt). |

Additional information:

* [Detailed instructions for zone-based fares use case.](gtfs-fares.md#zone-or-stop-based-fares)  
* [Full stops.txt documentation.](https://gtfs.mobilitydata.org/spec/gtfs-schedule#h.o3q8uyzc81o7)

### Fare rules: `fare_leg_rules.txt`

Once all the components of the fare policy are defined, how are they actually supposed to work together?

`fare_leg_rules.txt` unifies the policies between the transit network files (e.g. `routes.txt`, `stops.txt`) and other fare policy files (e.g. [fare_products.txt](gtfs-fares.md/#fare-products-fare_productstxt)) and assigns prices. This file can be quite long.

`Fare Leg`

:  A single trip on a vehicle. A leg has a start point, an end point, and takes place on a route. A traveler’s journey may involve one or more legs. A trip with two legs is said to require a “transfer” between legs.

`Fare Leg Group`

:  A leg_group_id can identify a set of legs with the same fare rules. This field is only required when transfers to or from the leg group are discounted.  

   For example, there may be several ways to pay for a ride on a local bus (adult fare, senior fare, etc.). Regardless of how the rider pays for that bus ride, they can board a second bus within 120 minutes for free. Grouping all the eligible leg rules into a single “leg_group_id” makes it possible to refer to them easily in the fare_transfer_rules.txt file.

??? info "Fields in fare_leg_rules.txt"  

    | <div style="width:150px">Field</div> | Description                                                                                                                                                                                                                                                                                                                                                                              |
    | :----------------------------------- | :--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
    | `leg_group_id`                       | Only fill out this field if it’s needed to define rules in `transfer_rules.txt`. For example: Riders can transfer from express routes to local for free, but pay a surcharge to transfer from local to express. Naming all the rules for local routes as “local” and express as “express” makes it possible to describe these transfers succinctly. This name does not appear to riders. |
    | `fare_leg_name`                      | This field is optional, unless a `fare_product_id` is defined, in which case it is forbidden.    This is a human-readable name that can be shown to users to describe the route. “Local” and “Express” are still good examples.                                                                                                                                                          |
    | `network_id`                         | If the fares are based on route, this is the field that links the fare rule to a set of routes. Otherwise, this field should be blank.                                                                                                                                                                                                                                                   |
    | `from_area_id`                       | If the fares are based on stop, this is the field that links the fare rule to the departing stops. Otherwise, this field should be blank. <br>See [Zone, stop, and area-based fares section](gtfs-fares.md#zone-stop-or-area-based-fares) for further discussion.                                                                                                                        |
    | `to_area_id`                         | If the fares are based on stop, this is the field that links the fare rule to the arriving stops. Otherwise, this field should be blank. <br>See [Zone, stop, and area-based fares section](gtfs-fares.md#zone-stop-or-area-based-fares) for further discussion.                                                                                                                         |
    | `contains_area_id`                   | If the fares are based on stop, and first/last stop aren’t sufficient to determine the fare, this field allows the fare rule to link to a set of stops along the route. Otherwise, this field should be blank. <br>See [Zone, stop, and area-based fares section](gtfs-fares.md#zone-stop-or-area-based-fares) for further discussion.                                                   |
    | `is_symmetrical`                     | This field is required when `from_area_id` and `to_area_id` are defined. Otherwise it should be blank. If the fares are the same when the “from” and “to” stops are reversed, the fare is considered “symmetrical.”  The options are: <br>0: Not symmetrical<br>1: Is symmetrical                                                                                                        |
    | `distance_type`                      | For distance-based fares, indicates if distance should be calcuated based on stops or crow-fly distance. See [distance-based fares section](gtfs-fares.md#distance-based-fares) for further discussion.                                                                                                                                                                                  |
    | `min_distance`                       | If the fares are distance-based, each set of distances with a unique price must be defined in its own row. This field and the next (max_distance) define the bounds of each range.                                                                                                                                                                                                       |
    | `max_distance`                       | See min_distance                                                                                                                                                                                                                                                                                                                                                                         |
    | `service_id`                         | Some fares only apply to certain days. GTFS uses service_id to define periods of time with different kinds of schedules. The most likely use of this field is to describe future fares. If fares are changing on January 1, for example, the calendar.txt file should define one service_id for dates through December 31 and another for dates starting January 1.                      |
    | `amount`                             | The cost of the fare leg. <br> Required field. Can be 0. <br> Just use the number. Don’t include a “$”                                                                                                                                                                                                                                                                                   |
    | `fare_product_id`                    | Fare leg rules must be written to show the price at boarding if the rider has a fare product. <br> The most common case is that the rider has some daily or monthly pass, which they already paid for, and makes the fare for their trip free. Another common case is when a rider has a local pass, but still needs to pay a surcharge for an express trip                              |
    | `fare_containers_id`                 | Fare leg rules must be written for each fare container.                                                                                                                                                                                                                                                                                                                                  |
    | `rider_category`                     | Fare leg rules must be written for each rider category.                                                                                                                                                                                                                                                                                                                                  |
    | `eligible_cap_id`                    | If fare capping is available, and the leg would contribute to that fare capping total, include the eligible_cap_id in this field.                                                                                                                                                                                                                                                        |

### Transfer rules: `fare_transfer_rules.txt`

fare_transfer_rules.txt defines fare rules and discounts for trips involving more than one leg in fare_leg_rules.txt.

??? info "Fields in fare_transfer_rules.txt"

    | Field               | Requirements                                                     | Comments                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                               |
    | :------------------ | :--------------------------------------------------------------- | :--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
    | from_leg_group_id   |                                                                  | Refers to the `fare_leg_rules.leg_group_id` referencing the leg is the rider transferring from.                                                                                                                                                                                                                                                                                                                                                                                                                        |
    | to_leg_group_id     |                                                                  | Refers to the `fare_leg_rules.leg_group_id` referencing the leg is the rider transferring to.                                                                                                                                                                                                                                                                                                                                                                                                                          |
    | is_symmetrical      |                                                                  | Indicates if the rules would be the same if the legs in to_leg_group_id and from_leg_group_id were switched. <br>Options are:<ul><li>0: not symmetrical</li><li>1: symmetrical.</li></ul>                                                                                                                                                                                                                                                                                                                              |
    | spanning_limit      |                                                                  | How many legs can this transfer rule be applied to? <br>Available options are:<ul><li>0: No limit.</li><li>2: After paying to board the first vehicle, the passenger can transfer to a second. Can be 3, 4, or more.</li></ul>                                                                                                                                                                                                                                                                                         |
    | duration_limit      | Optional                                                         | How long the transfer window is, in seconds. <br> For example: 3600 = one hour, 7200 = two hours, etc.                                                                                                                                                                                                                                                                                                                                                                                                                 |
    | duration_limit_type | Required if duration_limit is defined. Forbidden if it is empty. | "Defines from when the duration limit is counted. <br> Valid options are:<ul><li>0: From the time the rider boards the first leg to when they alight the last leg.</li><li>1: From the time the rider boards the first leg to when they board the last leg.</li><li>2: From the time the rider alights the first leg to when they board the last leg.</li><li>3: From the time the rider alights the first leg to when they alight the last leg.</li></ul>                                                             |
    | fare_transfer_type  | Required if the `amount` below is defined.                       | "How the transfer discount is applied. <br>Valid options are:<ul><li>0: The total cost of the multi-leg trip is the cost of the FIRST leg PLUS the amount in the next field.</li><li>1: The total cost of the multi-leg trip is the cost of ALL legs PLUS the amount in the next field.</li><li>2: The total cost of the multi-leg trip is the cost of WHICHEVER leg costs the most PLUS the amount in the next field.</li><li>3: The total cost of the multi-leg trip is JUST the amount in the next field.</li></ul> |
    | amount              | Required                                                         | "The cost of the transfer rule, including 0.                                                                                                                                                                                                                                                                                                                                                                                                                                                                           |
    | currency            | Required                                                         | USD in the United States.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                              |
    | fare_product_id     |                                                                  | If this rule is based on a fare product, include its ID here.                                                                                                                                                                                                                                                                                                                                                                                                                                                          |
    | fare_container_id   |                                                                  | If this rule is based on a fare container, include its ID here.                                                                                                                                                                                                                                                                                                                                                                                                                                                        |
    | rider_category_id   |                                                                  | If this rule is based on a rider category, include its ID here.                                                                                                                                                                                                                                                                                                                                                                                                                                                        |
    | eligible_cap_id     |                                                                  | If the transfer amount counts towards a fare cap, include its ID here.                                                                                                                                                                                                                                                                                                                                                                                                                                                 |

??? tip "Explicitly code cash or paper transfers"

    Any regular fare is NOT assumed to contain a free transfer. A regular fare only contains the fare. Therefore, transfers must be defined when they are an add-on/perk.

    Transfers SHOULD be explicitly defined in `fare_transfer_rules.txt` when they are considered a special bonus of paying a one-way fare.

    Imagine the following scenario: A rider would board Bus A, pay their fare in cash, and receive a paper transfer ticket from the bus driver for use when they transfer to and board Bus B.  In this scenario, defining a free transfer tells the app that the rider received a paper transfer ticket.

??? tip "Passes assume free transfers"

    Any time-based pass (daily, weekly, monthly, etc.) is assumed to offer unlimited rides. Example: Using a Santa Monica day pass to transfer from Santa Monica Route 1 to Santa Monica Route 2.

    This does not apply to fare booklets. Example: a Caltrain 10-ride booklet.

    Makes sure all the passes accepted for payment should be listed in the `fare_leg_rules.txt`.

??? tip "Code symmetrical transfer behaviors once"

    If a transfer has the same behavior in either directions (e.g. Agency A -> Agency B is the same as Agency B -> Agency A), write only one transfer record and indicate this behavior with `is_symmetrical` = 1.

    This only applies when the transfer rules, including cost, are the same forward as they are backward.

    CAUTION: Many regionally connected agencies may offer free transfers TO an agency, but that other agency does not offer free transfers in reciprocation. In this case, only transfers from Agency A to Agency B would be defined in Agency B's feed, with `is_symmetrical` = 0.

??? example "fare_transfer_rules.txt for Nevada County Connects"
    `fare_transfer_rules.txt` (excerpt):

    | from_leg_group_id | to_leg_ group_id | is_symmetrical | spanning_limit | duration_limit | duration_type | fare_transfer_type | amount | currency |
    | :---------------- | :--------------- | :------------- | :------------- | :------------- | :------------ | :----------------- | :----- | :------- |
    | 1-1               | 1-1              | 0              | 2              | 7200           | 2             | 1                  |        |          |
    | 1-1               | 1-2              | 1              | 2              | 7200           |               |                    | 1.5    | USD      |
    | 1-1               | 2-2              | 1              | 2              | 7200           |               |                    | 1.5    | USD      |
    | 1-2               | 1-2              | 0              | 2              | 7200           |               |                    |        |          |
    | 1-2               | 2-2              | 1              | 2              | 7200           |               |                    | 1.5    | USD      |
    | 2-2               | 2-2              | 0              | 2              | 7200           |               |                    |        |          |

## Handling specific fares use cases

| Use Case                                                     |                                                                                               |
| :----------------------------------------------------------- | :-------------------------------------------------------------------------------------------- |
| [Free Fares](gtfs-fares.md#free-fares)                       | No fares should still be explicitly coded in order to differentiate from missing information. |
| [Route-based](gtfs-fares.md#route-based-fares)               | Defining special fares by route such as premium routes or free shuttles.                      |
| [Zone-based](gtfs-fares.md#zone-or-stop-based-fares)         |                                                                                               |
| [Distance-based](gtfs-fares.md#distance-based fares)         |                                                                                               |
| [Passes + Transit Cards](gtfs-fares.md#passes-transit-cards) |                                                                                               |

### Free Fares

If there are no fares, you only need `fare_leg_rules.txt` to describe the fare policy, and only one fare rule. Because this trip is free, leave the amount and currency fields blank.

??? example "Calaveras Transit"

    `fare_leg_rules.txt`:

    | `leg_group_id` | `amount` | `currency` |
    | :------------- | :------- | :--------- |
    | Calaveras      |          |            |

### Route-based fares

Route-based fares are represented in [GTFS-Fares](https://bit.ly/gtfs-fares) in two places:  

* [`routes.txt`](gtfs-fares.md#differentiate-fares-across-routes-in-routestxt): `network_id` specifies which routes belong to the network.
* [`fare_leg_rules.txt`](gtfs-fares.md#describe-transfer-rules-using-fare_transfer_rulestxt): `network_id` is referenced for the fare rule that applies to that group of routes.

??? example "Higher express route fares"
    In this example, the 95X Express route costs more.  

    To differentiate which routes are express or not, `routes.network_id` assigns each route to either “local” or “express”.

    | route_long_name                     | route_short_name | network_id |
    | :---------------------------------- | :--------------- | :--------- |
    | Walnut Creek BART/Broadway Plaza    | 4                | local      |
    | Pleasant Hill BART/Shadelands       | 7                | local      |
    | San Ramon/BART Walnut Creek         | 95X              | express    |
    | Walnut Creek BART/John Muir Med Ctr | 301              | local      |

    `fare_leg_rules.txt` assigns the proper fare to each `network_id`.

#### Route-based passes

If a bus pass is used in `fare_leg_rules.txt`, the `network_id` must either be:

* blank: if pass can be used on ALL networks  
* have an entry for EVERY network it is valid for  

??? example "Separate commuter and local passes"
    The El Dorado Transit commuter pass works for local and commuter networks, but the local pass only works on local routes.

    (1) Articulate which routes belong to which network in `routes.network_id`.

    (2) In `fare_leg_rules.txt`, represent the two networks which the commuter pass can access by either:

     - leave `network_id` blank   OR
     - define two entries – one for each `network_id`:

     | fare_product_id | network_id |
     | :-------------- | :--------- |
     | commuter_pass   | commuter   |
     | commuter_pass   | local      |

     Both options are valid and tell the consuming application that the Commuter Pass can be used on both networks.

### Zone or stop-based fares

Stop-, or zone-based fares are used if the fare is based on where the rider boards/alights. Stop-based fares can be though of as a zone which only has a single stop.

Zone-based fares are represented in [GTFS-Fares](https://bit.ly/gtfs-fares) in two places:  

* stops.txt: area_id in dictates the stop, or group of stops (zone) that is applicable.
* fare_leg_rules.txt: from_area_id, to_area_id and optionally, contains_area_id link a fare rule to the zones in stops.area_id.

??? tip "Transitioning from Fares v1: zone_id vs area_id"

    - Fares v1: `stops.zone_id`, a number with attributes which can be looked up in `fare_zone_attributes`
    - Fares v2: `stops.area_id`, can be human-readable.

    `zone_id`=`area_id`, they are the same datapoint just with different names.  

    If both fares v1 and v2 are being published simultaniously, `area_id` must match 1:1 with `zone_id`.

=== "Origin and destination zones"

    ![](/img/contains_area_A.png){ align=left }
    Image shows the boarding and alighting stops, along with four fare zones: 1, 2, 3, 4.

    Leaving `contains_area_id` blank indicates that there is no "thru" requirement and any travel between zone 1 and 4 would be subject to the fare leg rule.

    - `from_area_id` = 1  
    - `to_area_id` = 4
    - `contains_area_id` = blank

=== "Requiring travel thru zone 2"

    ![](/img/contains_area_A.png){ align=left }
    Shows a route that is from zone 1 to zone 4 and would be subject to a fare leg rule where:

    - `from_area_id` = 1  
    - `to_area_id` = 4
    - `contains_area_id` = 2 or is blank, indicating that there is no "thru" requirement.

=== "Requiring travel thru zone 3"

    ![](/img/contains_area_B.png){ align=left }
    Shows a route that is from zone 1 to zone 4 and would be subject to a fare leg rule where:

    - `from_area_id` = 1  
    - `to_area_id` = 4
    - `contains_area_id` = 3 or is blank, indicating that there is no "thru" requirement.

??? example "Zone based fares - Nevada County Connects"

    Zones are defined in the `area_id` in `stops.txt`:

    | `stop_id` | `stop_name `                              | `area_id` |
    | :-------- | :---------------------------------------- | :-------- |
    | 2558674   | Zion St across from Rankins Trailer Park  | 1         |
    | 2558676   | Zion St at SPD Market and Seven Hills Ctr | 1         |
    | 2617950   | Broad Street @ City Hall                  | 2         |
    | 2617951   | Glenbrook Shopping Ctr                    | 2         |

    With two zones, all fares will have to be defined three times in `fare_leg_rules.txt`:

    * `fare_leg_name: 1-1`: Trips within zone 1
    * `fare_leg_name: 2-2`: Trips within zone 2
    * `fare_leg_name: 1-2`: Trips between zones 1 and 2

    | `leg_group_id` | `fare_leg_name` | `from_area_id` | `to_area_id` | `is_symmetrical` | `fare_product_id`  | `rider_category_id` | `amount` | `currency` |
    | :------------- | :-------------- | :------------- | :----------- | :--------------- | :----------------- | :------------------ | :------- | :--------- |
    | nevada         | 1-1             | 2371           | 2371         |                  |                    |                     | 1.5      | USD        |
    | nevada         | 1-1             | 2371           | 2371         |                  |                    | senior              | 0.75     | USD        |
    | nevada         | 1-1             | 2371           | 2371         |                  |                    | disabled            | 0.75     | USD        |
    | nevada         | 1-1             | 2371           | 2371         |                  | day_general_zone1  |                     |          |            |
    | nevada         | 1-1             | 2371           | 2371         |                  | day_senior_zone1   | senior              |          |            |
    | nevada         | 1-1             | 2371           | 2371         |                  | day_disabled_zone1 | disabled            |          |            |
    | nevada         | 2-2             | 2372           | 2372         |                  |                    |                     | 1.5      | USD        |
    | nevada         | 2-2             | 2372           | 2372         |                  |                    | senior              | 0.75     | USD        |
    | nevada         | 2-2             | 2372           | 2372         |                  |                    | disabled            | 0.75     | USD        |
    | nevada         | 2-2             | 2372           | 2372         |                  | day_general_zone1  |                     |          |            |
    | nevada         | 2-2             | 2372           | 2372         |                  | day_senior_zone1   | senior              |          |            |
    | nevada         | 2-2             | 2372           | 2372         |                  | day_disabled_zone1 | disabled            |          |            |
    | nevada         | 1-2             | 2371           | 2372         | 1                |                    |                     | 3        | USD        |
    | nevada         | 1-2             | 2371           | 2372         | 1                |                    | senior              | 1.5      | USD        |
    | nevada         | 1-2             | 2371           | 2372         | 1                |                    | disabled            | 1.5      | USD        |
    | nevada         | 1-2             | 2371           | 2372         | 1                | day_general_zone2  |                     |          |            |
    | nevada         | 1-2             | 2371           | 2372         | 1                | day_senior_zone2   | senior              |          |            |
    | nevada         | 1-2             | 2371           | 2372         | 1                | day_disabled_zone2 | disabled            |          |            |

### Distance-based fares

Some fares are based on absolute distance. This is different from stop-based fares, where certain stop combinations have specified prices.

=== "Distance types"

    ![Distance Type Example](/img/distance_type_ex.png){ align=left }
    Image shows the transit route as a grey arc with flags representing stops.

    The green flag labeled “board” is where the rider boards and starts their trip.

    It passes through several gray flagged-stops before the rider alights at the red flagged-stop.

=== "Stops-based"
    `distance_type = 0` indicates that distance should be calculated based on the number of stops the rider passes through.

    ![Distance Type Example](/img/distance_type_0.png){ align=left }

    !!! Example
        - Going 1 to 9 stops costs $1  
        - Going 10 to 15 stops costs $2

        `fare_leg_rules.txt`:

        | `distance_type` | `min_distance` | `max_distance` | `amount` |
        | :-------------- | :------------- | :------------- | :------- |
        | 0               | 1              | 9              | 1        |
        | 0               | 10             | 15             | 2        |

=== "Mileage-based"
    `distance_type = 1` indicates that distance should be the actual distance between the first and last stop, regardless of the path the vehicle takes.

    ![Distance Type Example](/img/distance_type_1.png){ align=left }

    !!! Example
        - Going up to 4 miles stops costs $1  
        - Going up to 15 miles stops costs $2

        `fare_leg_rules.txt`:

        | `distance_type` | `min_distance` | `max_distance` | `amount` |
        | :-------------- | :------------- | :------------- | :------- |
        | 1               | 0              | 4              | 1        |
        | 1               | 4              | 15             | 2        |

    Note that this table doesn’t define a unit, and it uses whatever unit is defined in `shapes.shape_dist_traveled`. This is usually feet, but is important to confirm.

### Passes + Transit Cards

#### Multi-Agency Passes - External to GTFS Dataset

Even if a pass can be used for transit routes outside of the routes defined in this particular GTFS Schedule Dataset, the additional routes shouldn't be added here - but rather similarly represented in their respective GTFS Schedule Dataset.

??? example "El Dorado and Sacramento RT Combo Pass."

    For El Dorado's fares dataset, only El Dorado's `routes.txt` file needs to be included.
    In this situation, the data will explain which El Dorado routes the pass can be used on.
    It will NOT define which Sac RT routes the pass can be used on, even though the pass can be used on them.
    This is okay.

    Although the data will not explicitly state which "outside agency" (Sac RT) routes this combo pass can be used on, if the fare product name is descriptive enough it should point riders in the right direction.  It implies:

    > This combo pass can be used for the Sac RT system, in some capacity, but I don't know which specific routes

#### Rider-based Cards

Many cards (container_ids) are issued based on rider type(rider_cateogory_id), like a "senior pass".  The association between these is made in fare_leg_rules.txt.

??? example "Glendora Senior, Disabled, and Student fares"
    Glendora Transportation Division charges fares They offer discounts to three rider categories (seniors, persons with disabilities, and students), and sell two kinds of passes. Both passes require using a TAP card, and different riders pay different costs for their TAP cards.

    Note that some fare products and containers are limited to certain rider categories. It isn’t required to list that `rider_category_id` again in the fare rule row, but is more complete.

    `fare_leg_rules.txt`:

    | leg_group_id | fare_product_id | fare_container_id | rider_category_id | amount | currency |
    | :----------- | :-------------- | :---------------- | :---------------- | :----- | :------- |
    | Glendora     |                 |                   |                   | 1      | USD      |
    | Glendora     |                 |                   | senior            | 0.5    | USD      |
    | Glendora     |                 |                   | disabled          | 0.5    | USD      |
    | Glendora     |                 |                   | student           | 0.75   | USD      |
    | Glendora     | 1-day_regular   | tap_regular       |                   |        |          |
    | Glendora     | 1-day_senior    | tap_senior        | senior            |        |          |
    | Glendora     | 1-day_disabled  | tap_disabled      | disabled          |        |          |
    | Glendora     | 30-day_regular  | tap_regular       |                   |        |          |
    | Glendora     | 30-day_senior   | tap_senior        | senior            |        |          |
    | Glendora     | 30-day_disabled | tap_disabled      | disabled          |        |          |

#### Passes with asymmetrical reciprocity"

Sometimes a premium pass or ticket is sold which can be used on a whole network whereas a discounted pass is only valid on part of the network.  This can be represented at either the route (network_id) or zone level so long as they are defined within the same GTFS dataset.

??? example "Santa Maria Area Transit Passes and Breeze Passes"

    Santa Maria, California offers discounts to riders and sells passes, but their fares vary by route. The Breeze pass is accepted on SMAT routes, but SMAT passes are not accepted on Breeze routes.

    `fare_leg_rules.txt`:

    | leg_group_id | network_id | fare_product_id         | rider_category_id | amount | currency |
    | :----------- | :--------- | :---------------------- | :---------------- | :----- | :------- |
    | SMAT         | SMAT       |                         |                   | 1.5    | USD      |
    | SMAT         | SMAT       |                         | senior            | 0.75   | USD      |
    | SMAT         | SMAT       |                         | disabled          | 0.75   | USD      |
    | BREEZE       | BREEZE     |                         |                   | 2      | USD      |
    | BREEZE       | BREEZE     |                         | senior            | 1      | USD      |
    | BREEZE       | BREEZE     |                         | disabled          | 1      | USD      |
    | SMAT         | SMAT       | SMAT_monthly_general    |                   |        |          |
    | SMAT         | SMAT       | SMAT_monthly_student    | senior            |        |          |
    | SMAT         | SMAT       | SMAT_monthly_disabled   | disabled          |        |          |
    | BREEZE       | BREEZE     | BREEZE_monthly_general  |                   |        |          |
    | BREEZE       | BREEZE     | BREEZE_monthly_senior   | senior            |        |          |
    | BREEZE       | BREEZE     | BREEZE_monthly_disabled | disabled          |        |          |
    | SMAT         | SMAT       | BREEZE_monthly_general  |                   |        |          |
    | SMAT         | SMAT       | BREEZE_monthly_senior   | senior            |        |          |
    | SMAT         | SMAT       | BREEZE_monthly_disabled | disabled          |        |          |

## Mechanics

### Importing existing GTFS

??? tip "When do I need to import from existing GTFS?"
    If either:  
    (1) Fares v2 files have already been coded and you are updating or modifying these files.  

    (2) Fares are a function of routes or zones/stops, which require editing existing `routes.txt` and `stops.txt` files respectfully.

This process is best done in a spreadsheet like Excel or Google Sheets.  

| Step                       | Instructions                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                     |
| :------------------------- | :------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 1. Acquire GTFS dataset    | Download or locate locally the zip folder with `routes.txt` and `stops.txt`.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                     |
| 2. Import into spreadsheet | <ol><li>Click File > Import > Text File</li><li>Select routes.txt and/or stops.txt from the downloaded zip folder</li><li>Click Delimited, then click Next</li><li>Under Delimiters, click Comma, unclick Tab, then click Next</li><li>Under Column Data Format, click Text<br> NOTE: You must select each column individually and change it to Text<br>NOTE: that the first three columns have been formatted to ‘Text’ while the rest have not</li><li>When you are done, click Finish</li><li>Now you will have imported the routes.txt and stops.txt data uncorrupted and ready for consuming applications upon export (If these steps are not followed, the validator will find errors before publishing, such as an invalid hex code of ‘0’ instead of ‘000000’)</li></ol> |
| 3. Add relevant data.      | <ul><li>If coding route-based fares, add and fill ‘network_id’ in `routes.txt`</li><li>If coding zone-based fares, add and fill ‘area_id’ column to `stops.txt`.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                 |
| 4. Export                  | Once the relevant routes and stops data has been added the spreadsheets will be ready for export. See below for tips                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                             |

??? tip "Transitioning from Fares v1 to Fares v2? Copy area_id to zone_id"
    If the Fares v1 GTFS data exists already and has been built correctly, existing `zone_id` values can be directly copied to `area_id`.  

    The `area_id` column will be used in `fare_leg_rules` to craft stops-based fares.

### Export from a spreadsheet

Each tab in an agency’s spreadsheet (rider_categories, fare_products, etc.) must be saved as a separate file in order to be published. These will download as .CSV files, and their filenames must be manually changed to .TXT.

| Step                                | Instructions                                                                                                                                                                                                                                                                                                                                                                                                                                                                                              |
| :---------------------------------- | :-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 1. Delete empty columns and rows    | Before saving tabs as separate files, delete all empty columns and rows to ensure erroneous data isn’t lurking off screen. The solution is to select all below and select all right, right click, and select delete. The delete key on Mac keyboards does not work.                                                                                                                                                                                                                                       |
| 2. Unhide all columns               | Just in case.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                             |
| 3. Explicitly Select Column Formats | Excel and Google Sheets will try to simplify or round data unless the types are explicitly asserted.  See table below for field-specific instructions.                                                                                                                                                                                                                                                                                                                                                    |
| 4. Export                           | <ul><li>Navigate to a tab.</li><li>`Google Sheets`: Click File > Download > CSV</li><li>`Excel`: Click File > Save As > File Format > CSV (Note: there are multiple CSV options, choose the basic ‘Comma Separated Values’ option)</li><li>Change the name of the file to be just what is needed by GTFS, and change the format to .txt. For example, `“Fares - Truckee - rider_categories.csv”` becomes `“rider_categories.txt”`.</li><li>`Excel`: Click "save".</li><li> Repeat for all tabs.</li></ul> |

Field formatting for export:  

| <div style="width:160px">Field</div> | Format As        | Description                                                                                         |
| :----------------------------------- | :--------------- | :-------------------------------------------------------------------------------------------------- |
| `routes.route_color`                 | Format as text   | Six character-long hexcodes. e.g. 000000 is the code for black and cannot be shortened to simply 0. |
| `routes.route_text_color`            | Format as text   | Six character-long hexcodes. e.g. 000000 is the code for black and cannot be shortened to simply 0. |
| `stops.lat`                          | Unrounded floats |                                                                                                     |
| `stops.lon`                          | Unrounded floats |                                                                                                     |

## Validate GTFS Fares v2 Data

In order to be ingested by trip planners and used by riders, validate that the data you have created is consistent with the specification and doesn't raise any critical errors.

Transit App has developed and released an [open-source GTFS Fares v2 validator](https://github.com/TransitApp/gtfs-fares-v2-validator), which will indicate parts of the data that are inconsistent with the specification.

![fares validator command line photo](/img/fares-validator.png "Transit App Fares validator in use"){ align=left }
_An example of the validator finding two errors in a set of Fares v2 data._

??? tip "Install and setup GTFS Fares v2 Validator"
    1. Verify that Python 3 is installed on your machine by opening the command prompt (PC) or terminal (mac) and entering:

        ```bash
        python3
        ```

    If it is not found, install the [latest Python release](https://www.python.org/downloads/).
    2. **Download** Transit App’s Fares V2 Validator from [GitHub](https://github.com/TransitApp/gtfs-fares-v2-validator).<br> Under the Code dropdown, there is an option to download the validator as a zipped folder.  
    2. Unzip and place the folder on your machine in a place that can be easily accessed such as your desktop or downloads folder.  

??? tip "Using GTFS Fares v2 Validator"

    1. Change the active directory or your command prompt (PC) or terminal (Mac) to the downloaded Fares V2 validator by using the command:

        ```bash
        cd [path where validator downloaded]
        ```  

        Example:
        ```bash
        cd C:\Users\S152973\Desktop\gtfs-fares-v2-validator-master
        ```
    2. Validate fares v2 data by entering the following command prompt (PC) or terminal (Mac):

        Validate command:
        ``` bash
        python3 validate.py [path to folder containing fares v2 data] -o report.txt
        ```

        Example:
        ``` bash
        python3 validate.py C:\Users\S152973\Desktop\Fares_v2 -o report.txt
        ```
    3. `report.txt` in GTFS Fares V2 Validator folder contains all errors found in the GTFS fares v2 data.

## QA/QCing GTFS Fares v2 Data

Fares v2 Data should be QA/QCed to make sure it accurately reflects fares.

The transit provider is the source of truth for their fare rules. Manually confirm that the rules in the fares data match the provider’s website, and then schedule time to step through the data and confirm with the vendor.

## Publishing Fares v2 in a GTFS Feed

Once you have the fares files coded, you need to include the TXT files in the .zip file containing the rest of the transit provider’s GTFS data.

=== "Self-publishing GTFS data"

    The process is relatively simple: add the new files in the zip file and post.

    !!! warning "Don't overwrite more up-to-date data in routes or stops"
        If you use routes.txt or stops.txt to represent fares, take care to align the most accurate information and not overwrite either the fares data or the schedule data.

=== "Vendor-published GTFS data"

     The following vendors can and will publish your GTFS Fares v2 Data using their existing GTFS data pipelines:

     * Trillium Solutions
     * Interline

    The following vendors are working to include GTFS Fares v2 data in their data pipelines:

     * GMV/Synchromatics  
     * Optibus  
     * Trapeze

    Don’t see the vendor that publishes your GTFS data on the list?

     * Check with us to see if we are already talking with them: [gtfsrt@dot.ca.gov](mailto:gtfsrt@dot.ca.gov)
     * Ask them!  Most vendors have been willing to adjust to clients’ needs for this.

## Example Fares v2 Datasets

Here are some GTFS datasets with some full examples with various use-cases for you to peruse.

| **Use Case**                  | **Example Dataset**                                                                                                                 |
| :---------------------------- | :---------------------------------------------------------------------------------------------------------------------------------- |
| Free Fares                    | [Truckee Tahoe Area Regional Transit](http://data.trilliumtransit.com/gtfs/laketahoe-ca-us/laketahoe-ca-us.zip)                     |
| Simple flat fares             | [Placer County Transit](http://data.trilliumtransit.com/gtfs/placercounty-ca-us/placercounty-ca-us.zip)                             |
| Fares by rider-type           | [Eastern Sierra Transit Authority](http://data.trilliumtransit.com/gtfs/easternsierra-ca-us/easternsierra-ca-us.zip)                |
| Multi-ride passes             | [Big Blue Bus](http://gtfs.bigbluebus.com/current.zip)                                                                              |
| Fares by payment method       | [Humboldt Transit Authority](http://data.trilliumtransit.com/gtfs/humboldtcounty-ca-us/humboldtcounty-ca-us.zip)                    |
| Distance-based fares          | [Monterey Salinas Transit](https://docs.google.com/spreadsheets/d/17G0vjBJA4YHR1mYBttIh-BEl_B6JCUQcI1K-ZEovZyg/edit#gid=2135765071) |
| Zone-based fares              | [Nevada County Connects](http://data.trilliumtransit.com/gtfs/goldcountrystage-ca-us)                                               |
| Contactless bank card payment | [Clean Air Express](http://www.cleanairexpress.com/GTFS/GTFS.zip)                                                                   |
| Transfer Rules                | [Nevada County Connects](http://data.trilliumtransit.com/gtfs/goldcountrystage-ca-us)                                               |
| Fares by route-type           | [Santa Maria Area Transit](http://data.trilliumtransit.com/gtfs/smat-ca-us/smat-ca-us.zip)                                          |
| Peak Pricing                  | WMATA                                                                                                                               |

## Additional Resources

What resources exist to help with Fares v2?

| **Item**             | **Description**                                                                                                                                                                                                                                                                |
| :------------------- | :----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Spreadsheet Template | Created by Cal-ITP, the [Fares - Template](https://docs.google.com/spreadsheets/d/1buRntvL6aItm8yXiJ_eEVpejUyWFeJ78flrrAFhHZX8/edit?usp=sharing) can help organize the required fields and files and can be easily exported to CSV TXT files to be included in a GTFS Dataset. |
| Validator            | Until the [Canonical GTFS validator]("https://github.com/MobilityData/gtfs-validator") is able (expected 2021 Q4), you can validate Fares v2 using the Python-based [open-source validator](https://github.com/TransitApp/gtfs-fares-v2-validator) released by Transit App.    |

## Where can I get help?

If the transit service you want to share fares for is in California, Caltrans staff can help you code, troubleshoot, and publish your data.  Inquire at [gtfsrt@dot.ca.gov](mailto:gtfsrt@dot.ca.gov).

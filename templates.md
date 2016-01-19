
#Organizing Business Logic Using the Template Design Pattern

Every Apex developer knows the feeling when a once elegant Apex class, as new business logic is implemented, gets bigger, less readable and more messy. Spagetty code, simply said.

We have had the very same problem with out Stripe for Salesforce app[http://]. This is an all-in-one payment gateway management application, so probabli the most important bit here is to manage the customers' cards.

Till we had only one simple businesss logic - *Charge a Card*, it was very simple. One class, some initialization logic, and an *execute()* method. It looked also OK when we added some more cool Stripe-provided features, like creating new customers when creating a new card, or adding a second card to an existing customer.

But soon it looked something like this:
##### PaymentMethodService()
* createCustomer()
* addCard()
* addCardToCustomer()
* addCardAndCharge();
* addCardAndChargeExistingTransaction();
* addCardToCustomerAndCharge();
* matchCustomersByAccount();
* matchCustomerByContact();


In was still manageable, but after this Stripe introduced the (absolutely great!) ACH payments.



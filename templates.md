
#Organizing Business Logic Using the Template Design Pattern

Every Apex developer knows the feeling when a once elegant Apex class, as new business logic is implemented, gets bigger, less readable and more messy. Spagetty code, simply said.

We have had the very same problem with out Stripe for [Salesforce app](http://). This is an all-in-one payment gateway management application, so probabli the most important bit here is to manage the customers' cards.

Till we had only one simple businesss logic - *Charge a Card*, it was very simple. One class, some initialization logic, and an *process()* method. It looked also OK when we added some more cool Stripe-provided features, like creating new customers when creating a new card, or adding a second card to an existing customer.

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


In was still manageable, but after this Stripe introduced the (absolutely great!) [ACH payments](https://support.stripe.com/questions/accepting-ach-payments-with-stripe). So now we need to deal all the use cases above, but not only in credit card context, but also with charging ACH Bank Accounts. This is at least 5 new method (all the class methods, but now with Bank Accounts). Also we decided to create some new use cases, as public facing checkouts, which works in *slightly different way*.

And this is the point here : *slightly different*. If we look on the big picture, credit cards or a bank accounts are esentially the sames: they are only payment methods, *implemented differently*.

So what about an architecture like this?

* build payment method processor()
* process payment method()

And where payment method processor could choose from the behaviours like this:
* createCustomer()
* lookupExistingCustomer()
* createCard()
* createBankAccount()
* charge()

Combining these behaviors we can cover all the possible payment method use cases. And if something new cames up, like BitCoin processing ([it is possible with Stripe](https://stripe.com/bitcoin), in fact), we just need add a new behavior method, called *createBitcoin()*.

It sounds like a good use case for **Template Design Pattern**.

> In software engineering, the template method pattern is a behavioral design pattern that defines the program skeleton of an algorithm in a method, called template method, which defers some steps to subclasses. ([Wikipedia](https://en.wikipedia.org/wiki/Template_method_pattern))

And it is very easy to implement in Apex. All we need is an *abstract template class*, a couple of *behavioral classes* and the *concrete implementations* of the algorithms.

```
	public abstract class PaymentMethodTemplate implements IPaymentMethod{
		private NewPMBehavior buildBehavior;
		private NewPMBehavior processBehavior;
		private NewPMBehavior afterActionBehavior;
        private Payment_Method__c pm;
		
		public NewPaymentMethodTemplate(Payment_Method__c pm) {
		    this.pm = pm;
		}

		public void execute() {
			this.buildBehavior.execute();
			this.processBehavior.execute();
			this.afterActionBehavior.execute();
		}
	}
	
	public abstract class NewPMBehavior {
		public Payment_Method__c pm;
		public NewPMBehavior(Payment_Method__c pm) {
			this.pm = pm;
		}
		public abstract void execute();
	}
```

So we are almost done -- there are 2 points here.
* The template class only one public method *execute()* which is responsible to run the behaviors in the right order -- it means that the caller method actually has no idea what is going on inside. It just knows that it's s generic payment method processor, which does the right thing with the right payment method.
* The template should share some date between the behaviours. In our case, it's the Payment Method. So, the object not only passed to the concrete implementation of the Template, but also to the behavioral classes. This way, an action made by the first behavior is visible to the second one, and so one.

Ok, so let's see a some concrete implementations.

```
	public class ProcessNewBankAccount extends NewPMBehavior {
		public ProcessNewBankAccount(Payment_Method__c pm) {
			super(pm);
		}

		public override void execute() {
			this.pm = newBankAccount(this.pm);
		}
	}
	
	public class BankAccountToCustomer extends PaymentMethodTemplate {
		public BankAccountToCustomer(Payment_Method__c pm) {
			super(pm);
			this.buildBehavior = new LookUpPossibleCustomers(pm);
			this.processBehavior = new ProcessNewBankAccount(pm);
			this.afterActionBehavior = new ChargeTransaction(pm);
		}
	}
```

Again, on the complete implementatons we are sharing the data which belongs to the main template class instanec. This way, the behavioral classes can made subsequent changes on the very same object. In the case above, we are looking up a possible customer record in the system, processing a new bank account and finally, once the payment method has all the required data in Salesforce, charging the customer.

So, the key points about using Template Pattern in Apex code:
*

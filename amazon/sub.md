well let's have a look at workflow

I'm sure most of you have bought something on Amazon at some stage or something similar.
when you buy something on Amazon you start a workflow.
so you say place your order first of all, 

【图1】

and then you might select your shipping address. 
you know where you can actually get this way actually going to receive this book whatever is the ordering

【图2】

you put in your credit card details and etc

【图3】

etc

【图4】

etc

【图5】

until you actually place your order.
now Amazon is not a monolith.
it's actually probably one of the most service-oriented companies in the world.
there is not an one terabyte Amazon back see that they roll out its a bunch of different services.
so you can imagine there's a lot of services going into this workflow which are orchestrating together to make this happen.
now if we look at the events or look at the backend messaging and events that happen when you place your order.
it might look something like this.
so we have here a sales service a finance service and a shipping service.
when we click that button we can send the command to the sales service to say place your order.
sales can then go and do whatever logic it needs to do:
have we still got this item in stock,
it's still for sale if it's not in stock can we order more etc etc
and in the end it says ok we're good to go.
the orders been placed it can publish an event which Finance and shipping can subscribe to.
finance when it receives that event it can bill the customer for that order.
it can raise an order billed event.
and then when shipping knows that the orders been placed the orders been billed we're good to go.
ship out the door raise another event.
you know an auto ship with something else then might want to carry on within that workflow.


the question then is what should each of these events contain.
should the order placed event contain all the information that shipping needs in order to ship the item so the address etc etc
should the event contain all the information that finance needs to complete the order to build the customer credit card details and that kind of thing 
now that would work, right, that would work 
but what's the problem with that approach
its coupling
we're getting back into a highly coupled situation
if that event needs to have the address the shipping option the credit card details and everything on it.
we can no longer make changes in an isolated fashion.
we want to change anything about how we ship things, or how we build for things.
we're gonna have to make changes in sales as well as finance, or sales as well as shipping.
imagine if we wanted to start charging in Bitcoin, 
we'd have to give sales all the information about how to charge in Bitcoin all the data that's required
so it could pass on that information to finance.
what if we wanted start shipping ebooks we don't send an e-book to a physical address that will just be weird.
so we don't want to give that we don't want to give sales all information about how should be an e-book 
so it can pass that on to shipping

now what I just described our fat events right
and you can see that if we were to start ship if we were to start shipping ebook
and charging a Bitcoin that event will get fatter and fatter and fatter, with more information on it
something's allowed to be null but other things must not be null 
when they're null and gets really complicated
what we want to do is have a look at how we can get those fat events back down to thin events
with not much information on them
a much more loosely coupled system


what if we could get it down we could get those events down to just an ID
so what if we said what if sales said order 1 2 3 has been placed
finance says right order 1 2 3 I know how to charge for that order
I'm going to charge a credit card or I'm going to start a Bitcoin transaction 
we then say order 1 2 3 has been placed to shipping order
1 2 3 has been billed to shipping an order
and at that point shipping can say right order 1 2 3 I'm going to ship that to a physical address
I'm going to send an email now
that would work fine as well


but we've got a problem here if sales is responsible for creating orders
so sales is the only thing that can say yes we can feel the fulfill this order 
at that point it will create an order object with an ID
and it will save it to its database
now how can finance possibly know about that order before it's been created
how can finance know what to do for order 1 2 3
if we're not creating the order ID 1 2 3 until we create the order in sales
it's a chicken egg scenario
but the thing is what we've said so far is we're going to create that order at the end of the workflow right
so we're saying we're creating it when we place the order
what if we turn that upside down and we create the ID right at the beginning 
we can do it right here

【图】

proceed to the checkout
and you can do this client side as well
you can create a good client side uuid
what this allows us to do is when we go into the rest of the workflow 
we can say write shipping here's the address and the order ID is 1 2 3
because we've already for the order ID when we said when we said place the place order 
when we go to finance we can say right here the credit card details for order 1 2 3
etc etc until you get to the end of the workflow 
that's when you send that command to sales to say place order
now if you want to if you want to introduce something like Bitcoin payment
we can make changes just in the finance service
because when finance service gets order 1 2 3
it can say ah I know how to charge wrist
it's either a credit card or its Bitcoin
and I can do the appropriate thing
similarly in shipping we can now ship ebooks without having to make changes in sales
we can do that isolated just in shipping
and the one weird trick that and they want us to do that was setting the ID at the front instead of at the beginning
now you might be thinking what if we tell the system where to ship our order 
we tell them how to charge rear
but the last minute your spouse walks through a door 
and they say ah now I'd better not all of this that we mad 
so you cancel the order 
you're closed a browser 
shipping is going to be sitting there with a whole bunch of addresses
which is never going to ship to 
finance gonna is going to be sitting there with a whole bunch of a bunch of payment details
which is never going to charge for
isn't that redundant data
isn't someone gonna have to come along and clean this data up 
we're gonna be holding all that junk right
but is it really redundant so
some businesses would they would kill for that kind of information 
they want to know who's selecting products and where they want into the shipment to 
but then don't complete their order right
because then you can give that to kind of your design team and your workflow team
and you can say well how can we make this better so that more people actually go and create
actually go and fint finalize the order
so it can actually be really really useful information 
you can get rich business data analytics from that incomplete order information storage
you know it's gonna be a row in a database
you know who really cares about how much that's gonna cost
but ultimately you don't really have to keep it in a database
either so some services might choose to hold it in some kind of session state that might be more applicable for finance 
you don't necessarily want to keep credit card details sitting in a database
you might choose to hold them in session state so that will go away if the orders cancel naturally

now so so what is it why why is it that we end up in this big ball of mud scenario
if we can do things like we just described
we're only sharing IDs
well it was this here right that's where the problem started
it was the sharing of data 
as soon as we started to share data between services
it it got this kind of ball rolling bounced down 
and it got us kind of moving towards that that possible end of that big ball of mud scenario
so this is what we want to avoid
want to avoid sharing data between services 
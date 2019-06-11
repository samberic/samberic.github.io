---
id: 112
title: 'How to &#8220;Maker&#8221; you tests clear'
date: 2015-03-21T09:18:52+08:00
author: Sam
layout: post
guid: http://samatkinson.com/?p=112
permalink: /how-to-maker-you-tests-clear/
dsq_thread_id:
  - "3613514492"
categories:
  - Uncategorized
tags:
  - testing
---
Tests should be your primary means of documentation in a system. I like to think we’ve all moved passed the idea of using comments in code (except for APIs), and most of us know to strive for clean, self documenting code. But for me tests are the easiest and most powerful way of documenting a section of code. If I’m looking at code and having a WTF moment, I should be able to flip to the tests and see what the intended behaviour was.

The problem is that, despite good intentions, writing clear and well documented tests is **hard**. Really hard. Particularly if you’re working with legacy code which has tight coupling, making it hard to split out the bits that you care about.

I recently discovered a test when going through a system which looked not unlike this:{% raw %}
<pre class="EnlighterJSRAW" data-enlighter-language="java">@Test
public void bankTransferWillIncreaseDestinationBankAccount() throws Exception {
    BankAccount bankAccount = new BankAccount(20.0);
    final BankTransfer mock = mockery.mock(ConcreteBankTransfer.class);

    mockery.checking(new Expectations(){{
         allowing(mock).accountFrom(); will(returnValue(“accFrom”));
         allowing(mock).accountTo(); will(returnValue(“accTo”));
         allowing(mock).name(); will(returnValue(“Jon Smith”));
         allowing(mock).transferId(); will(returnValue(“143NMd24”));
         allowing(mock).ccy(); will(returnValue(null));
         allowing(mock).overdraftLimit();will(returnValue(200.0));
         oneOf(mock).amount(); will(returnValue(20.0));
     }
     });
    bankAccount.apply(mock);
    assertThat(bankAccount.amount(), is(40.0));
}</pre>{% endraw %}
*This is a dramatic reconstruction of the actual code. No programmers were hurt in the creation of this code.*

The thing is, there were three tests in the class, which all had this vomit of “allowing”. This code smells badly. Mocks should be used to mock *behaviour*. In this case the BankTransfer object is to all intents and purposes a  POJO. Bad behaviour.

But the bigger issue here is that it’s impossible to see what’s going on. Does the fact ccy is null matter? What about the transfer ID? Does the overdraft have something to do with it?

This test is woefully unclear. Even if we remove the mock abuse and replace with a real implementation it doesn’t help.{% raw %}
<pre class="EnlighterJSRAW" data-enlighter-language="java">@Test
public void bankTransferWillIncreaseDestinationBankAccount() throws Exception {
    BankAccount bankAccount = new BankAccount(20.0);
    final BankTransfer mock = new ConcreteBankTransfer(“accFrom”, “accTo”, null, 20.0, 200.0, “Jon Smith”, “143NMd24”);

    bankAccount.apply(mock);
    assertThat(bankAccount.amount(), is(40.0));
}</pre>{% endraw %}
Sure there’s less code, but I have no idea what each field in the constructor means. If anything it’s less clear now which fields matter.

This is where Maker’s come in. A Maker allows you to construct an object whilst saying “I don’t care about any values except these specific ones”. Let’s look at the final code before showing the implementation.{% raw %}
<pre class="EnlighterJSRAW" data-enlighter-language="java">private Maker aBankTransfer = a(BankTransferMaker.BankTransfer);

@Test
public void bankTransferWillIncreaseDestinationBankAccount() throws Exception {
    BankAccount bankAccount = new BankAccount(20.0);
    BankTransfer bankTransfer = make(aBankTransfer.but(with(BankTransferMaker.amount, 20.0)));
    bankAccount.apply(bankTransfer);
    assertThat(bankAccount.amount(), is(40.0));
}</pre>{% endraw %}
For me this is infinitely clearer. The code does exactly as it says; it creates a BankTransfer (and we don’t care what the looks like), but we specify that it must have an amount of 20.0 as this is the value that we care about for our test. Very terse, clear and also reusable. Anywhere that needs a BankTransfer object can reuse this.

To use Maker, you need to import Nat Pryce’s “make-it-easy” (Maven details at <a href="http://mvnrepository.com/artifact/com.natpryce/make-it-easy">http://mvnrepository.com/artifact/com.natpryce/make-it-easy</a>). Then it’s simply a matter of creating your Maker.

There’s a fair amount of boilerplate code, and it can be quite tiresome to build a Maker. As a result you may want to think carefully before starting to use them everywhere.{% raw %}
<pre class="EnlighterJSRAW" data-enlighter-language="java">public class BankTransferMaker {

    public static final Property&lt;BankTransfer, String&gt; accountFrom = newProperty();
    public static final Property&lt;BankTransfer, String&gt; accountTo = newProperty();
    public static final Property&lt;BankTransfer, String&gt; name = newProperty();
    public static final Property&lt;BankTransfer, String&gt; transferId = newProperty();
    public static final Property&lt;BankTransfer, Double&gt; overdraftLimit = newProperty();
    public static final Property&lt;BankTransfer, Double&gt; amount = newProperty();
    public static final Property&lt;BankTransfer, ConcreteBankTransfer.Currency&gt; ccy = newProperty();

    public static final Instantiator BankTransfer = new Instantiator() {

       @Override public BankTransfer instantiate(PropertyLookup lookup) {

            BankTransfer bankTransfer = new ConcreteBankTransfer(
                lookup.valueOf(accountFrom, random(5)),
                lookup.valueOf(accountTo, random(5)),
                lookup.valueOf(ccy, new ConcreteBankTransfer.Currency(random(3))),
                lookup.valueOf(amount, nextDouble(0,100000)),
                lookup.valueOf(overdraftLimit, nextDouble(0,100000)),
                lookup.valueOf(name, random(10)),
                lookup.valueOf(transferId, random(10)));
  
            return bankTransfer;
           }
       };
}</pre>{% endraw %}
For each constructor parameter we need we create a Property value, which we have to type correctly. This is where a lot of the frustration can come from, as you have to manually build up this mapping.

We then create an Instantiator; this is how our object is actually created. You can set the default values to whatever you want; I’ve used apache-commons to plug random values in, because I really don’t care what’s there.

When it comes to creating test objects you can then modify any and all of these with the builder-style pattern seen in my original code base. We can change multiple values too, like so:
<pre class="EnlighterJSRAW" data-enlighter-language="java">make(aBankTransfer.but(
with(BankTransferMaker.amount, 20.0),
with(BankTransferMaker.transferId, “Octopus”)));</pre>
It’s a really nice way to give clear visibility to which values matter in your test.
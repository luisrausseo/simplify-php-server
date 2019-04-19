Simplify Payment PHP Server
=========================
This is an companion application to help developers start building mobile applications using Simplify Commerce by MasterCard to accept payments. For more information on how Simplify Commerce works, please go through the overview section of Tutorial at Simplify.com -  https://www.simplify.com/commerce/docs/tutorial/index

##Steps for running

* Register with Heroku (if you haven't done so already)
* Register with [Simplify Commerce](https://www.simplify.com/commerce/login/signup) and you will need API Keys (Under Settings/API Keys) in the next step

##Steps for integrating with iOS & Android apps



### Android:
```java
    final RequestQueue MyRequestQueue = Volley.newRequestQueue(this);
	simplify = new Simplify();
	simplify.setApiKey(publicAPIkey);

	mRef.addValueEventListener(new ValueEventListener() {
		@Override
		public void onDataChange(@NonNull DataSnapshot dataSnapshot) {
			String number = dataSnapshot.child("payment").child(Integer.toString(cardSelected)).child("number").getValue(String.class);
			String exp_month = dataSnapshot.child("payment").child(Integer.toString(cardSelected)).child("exp_month").getValue(String.class);
			String exp_year = dataSnapshot.child("payment").child(Integer.toString(cardSelected)).child("exp_year").getValue(String.class);
			String CVC = dataSnapshot.child("payment").child(Integer.toString(cardSelected)).child("CVV").getValue(String.class);
			String zipcode = dataSnapshot.child("payment").child(Integer.toString(cardSelected)).child("zipcode").getValue(String.class);
			final String payID = dataSnapshot.child("id").getValue(String.class);

			myCard = new Card()
					.setNumber(number.replace(" ",""))
					.setExpMonth(exp_month)
					.setExpYear(exp_year.substring(exp_year.length() - 2))
					.setCvc(CVC)
					.setAddressZip(zipcode);

			paymentStatus = number.replace(" ","") + "|" + exp_month+ "|" + exp_year+ "|" + CVC+ "|" + zipcode;

			// tokenize the card
			simplify.createCardToken(myCard, new CardToken.Callback() {
				@Override
				public void onSuccess(final CardToken cardToken) {
					try {
						System.out.println(cardToken.getId());
						String url = "http://ezservepayment.herokuapp.com/charge.php";
						StringRequest MyStringRequest = new StringRequest(Request.Method.POST, url, new Response.Listener<String>() {
							@Override
							public void onResponse(String response) {
								progressDialog.cancel();
								try {
									JSONObject jsonObject = new JSONObject(response);
									Toast.makeText(PayForItems.this, response, Toast.LENGTH_LONG).show();
									if (jsonObject.get("status").toString().equals("APPROVED")) {
										startActivity(new Intent(PayForItems.this, CustomerMainActivity.class).setFlags(Intent.FLAG_ACTIVITY_CLEAR_TOP));
										//TODO: clear table and add it to customer's bills
									}
								} catch (JSONException e) {
									//For debugging
									Toast.makeText(PayForItems.this, e.getMessage(), Toast.LENGTH_LONG).show();
								}
							}
						}, new Response.ErrorListener() { //Create an error listener to handle errors appropriately.
							@Override
							public void onErrorResponse(VolleyError error) {
								//This code is executed if there is an error.
							}
						}) {
							protected Map<String, String> getParams() {
								Map<String, String> MyData = new HashMap<String, String>();
								MyData.put("simplifyToken", cardToken.getId()); //Send the card token with the request
								MyData.put("amount", String.valueOf(total*100)); //send the amount in cents
								MyData.put("customer", payID); //send the customer id
								return MyData;
							}
						};

						MyRequestQueue.add(MyStringRequest);
					} catch (Exception e) {
						Toast.makeText(PayForItems.this, e.toString(), Toast.LENGTH_LONG).show();
					}
				}
				@Override
				public void onError(Throwable throwable) {
					Toast.makeText(PayForItems.this, throwable.getMessage(), Toast.LENGTH_LONG).show();
				}
			});

		}
		@Override
		public void onCancelled(@NonNull DatabaseError databaseError) {
			paymentStatus = "Error 001";
		}
	});

```

##References
* https://www.simplify.com/commerce/docs/examples/heroku
* https://www.simplify.com/commerce/docs
* https://www.simplify.com/commerce/docs/tutorial/index#payments-form
* https://www.simplify.com/commerce/docs/tutorial/index#testing
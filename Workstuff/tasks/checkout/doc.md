
website: 
	cart summary changes -> design changes will come from ved
	apply coupons logic -> this will change -> fetch coupons api required to get coupons
	- the logic of applying coupons is complex. there can be a lot of terms of conditions
	removal of shopify code 
	replace create-draft-order with the alternative that creates an order in OMS(confirm?), then we have to redirect to checkout ui

checkout ui
	there will be changes related to showing what coupons have been applied. 
	other things should work how it is already. 


---
admin dashboard:
- what things should show up
	- day pass isnt there rn
Flow of things: 



**booking conf room** :
	create-draft-order payload: https://orders-dot-wework-staging.el.r.appspot.com/create-draft-order
	```
```json
	
	{
	    "redirect_to": "http://localhost:3000/workspaces/on-demand/order-confirmation/?order_id=",
	    "utm_params": {
	        "referrer": "https://upflex-staging.myshopify.com/",
	        "point_of_contact_url": "http://localhost:3000/",
	        "lead_source_url": "http://localhost:3000/workspaces/on-demand/conference-room/bangalore/galaxy-residency-road/cart-summary/"
	    },
	    "customer_email": "krtest1996@gmail.com",
	    "notes": "NA",
	    "buildingId": "8992688275740",
	    "line_items": [
	        {
	            "variantId": "gid://shopify/ProductVariant/47107011641617",
	            "quantity": 1,
	            "type": "CONFERENCE_ROOM"
	        }
	    ],
	    "gstin_note": "GSTIN: ",
	    "draft_notes": {
	        "isDashboard": false,
	        "conference_rooms_info": [
	            {
	                "reservation_id": "jBoxUmClR8JERSK5uU9V",
	                "handle": "conference-room-credit",
	                "building_title": "Galaxy",
	                "booking_date": "2025-04-04T10:30:00.000Z",
	                "variant_title": 47107011641617,
	                "booking_start_time": "2025-04-04T09:30:00.000Z",
	                "member_count": 3,
	                "no_of_hrs": 1,
	                "conference_room_id": "EYNWwnBhpmBSh1oIETyH",
	                "city_name": "Bangalore",
	                "micromarket_name": "mg-road",
	                "building_id": "8992688275740",
	                "building_zone": "Bangalore"
	            }
	        ]
	    },
	    "useVoCredits": false,
	    "useBundleCredits": false,
	    "amountSavedAsLoyaltyMember": 0,
	    "amountCouldBeSavedAsLoyaltyMember": 50,
	    "userAnalytics": {
	        "customerID": "cabe54b0-5e1a-013d-8aa7-7a9c0bfc7f1c",
	        "name": "krtest1996",
	        "encryptedEmailId": "8ebddab62e2785414003dbbd3acecafd",
	        "encryptedMobileNumber": "",
	        "existingUser": "Yes",
	        "ErrorCode": "",
	        "isReferral": "No",
	        "numberOfReferrals": 2,
	        "clientID": "1494257004.1743681819",
	        "shopifyCustomerID": 8430820983057,
	        "item_brand": "Online",
	        "location_id": "ChIJx8LwiHsXrjsRPHXzKogknak",
	        "productInfo": {
	            "product": "conference room",
	            "building": "Galaxy",
	            "micromarket": "Mg Road",
	            "city": "Bangalore",
	            "created_at": 1743758532726
	        }
	    },
	    "bundleDiscountType": null,
	    "meeting_ids": "jBoxUmClR8JERSK5uU9V_EYNWwnBhpmBSh1oIETyH"
	}


```


graphql payload to upflex:
https://upflex-staging.myshopify.com/api/graphql

```json
{
    "operationName": "cartCreate",
    "variables": {
        "cartInput": {
            "note": "H6nY7q4Yw2oTu7fWPIZx",
            "attributes": [
                {
                    "key": "conferenceRoomInfo",
                    "value": "{\"conferences\":[{\"reservation_id\":\"jBoxUmClR8JERSK5uU9V\",\"handle\":\"conference-room-credit\",\"building_title\":\"Galaxy\",\"booking_date\":\"2025-04-04T10:30:00.000Z\",\"variant_title\":47107011641617,\"booking_start_time\":\"2025-04-04T09:30:00.000Z\",\"member_count\":3,\"no_of_hrs\":1,\"conference_room_id\":\"EYNWwnBhpmBSh1oIETyH\",\"city_name\":\"Bangalore\",\"micromarket_name\":\"mg-road\",\"building_id\":\"8992688275740\",\"building_zone\":\"Bangalore\"}]}"
                },
                {
                    "key": "requestId",
                    "value": "2h1olK6DVXMRWgsbsEhX"
                },
                {
                    "key": "redirectTo",
                    "value": "http://localhost:3000/workspaces/on-demand/order-confirmation/?orderObjectId=H6nY7q4Yw2oTu7fWPIZx&order_id="
                },
                {
                    "key": "orderCreatorEmail",
                    "value": "krtest1996@gmail.com"
                },
                {
                    "key": "isPartner",
                    "value": "true"
                },
                {
                    "key": "utmParams",
                    "value": "{\"referrer\":\"https://upflex-staging.myshopify.com/\",\"point_of_contact_url\":\"http://localhost:3000/\",\"lead_source_url\":\"http://localhost:3000/workspaces/on-demand/conference-room/bangalore/galaxy-residency-road/cart-summary/\"}"
                },
                {
                    "key": "userAnalytics",
                    "value": "{\"customerID\":\"cabe54b0-5e1a-013d-8aa7-7a9c0bfc7f1c\",\"name\":\"krtest1996\",\"encryptedEmailId\":\"8ebddab62e2785414003dbbd3acecafd\",\"encryptedMobileNumber\":\"\",\"existingUser\":\"Yes\",\"ErrorCode\":\"\",\"isReferral\":\"No\",\"numberOfReferrals\":2,\"clientID\":\"1494257004.1743681819\",\"shopifyCustomerID\":8430820983057,\"item_brand\":\"Online\",\"location_id\":\"ChIJx8LwiHsXrjsRPHXzKogknak\",\"productInfo\":{\"product\":\"conference room\",\"building\":\"Galaxy\",\"micromarket\":\"Mg Road\",\"city\":\"Bangalore\",\"created_at\":1743758532726}}"
                },
                {
                    "key": "amountCouldBeSavedAsLoyaltyMember",
                    "value": "50"
                },
                {
                    "key": "invoiceData",
                    "value": "{\"building\":{\"name\":\"Galaxy, Residency Road\",\"state\":\"Karnataka\",\"state_code\":\"KA\"}}"
                },
                {
                    "key": "locationUUID",
                    "value": "fdb15bdf-19e2-4abe-bd40-7b7fa9816fb5"
                },
                {
                    "key": "myhq_meta",
                    "value": "{\"building\":\"Galaxy, Residency Road\",\"building_name\":\"Galaxy\",\"building_id\":\"8992688275740\"}"
                }
            ],
            "buyerIdentity": {
                "email": "krtest1996@gmail.com"
            },
            "lines": [
                {
                    "merchandiseId": "gid://shopify/ProductVariant/47107011641617",
                    "quantity": 1
                }
            ]
        }
    },
    "query": "mutation cartCreate($cartInput: CartInput!) {\n  cartCreate(input: $cartInput) {\n    cart {\n      id\n      checkoutUrl\n      __typename\n    }\n    __typename\n  }\n}"
}
```
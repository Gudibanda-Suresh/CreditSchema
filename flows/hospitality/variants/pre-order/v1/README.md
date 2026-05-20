# pre-order — ION Hospitality Variant v1

**Applies to:** delivery-order  
**Activation:** `RestaurantOffer.deliveryMode IN [SCHEDULED, PRE_ORDER, BOTH]`  

Consumer books a delivery slot in advance. Kitchen prepares at the right
time, not immediately. Cancellation window is wide — closes
`preOrderLeadTimeHours` before kitchen start time.

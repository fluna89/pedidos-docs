# Functional Requirements — Customer Module

> Source: requirements specification, Section 2

## General Design Criteria

- **Mobile first** for customer and guest views (most users will order from their phone)
- **Desktop first** for the admin panel (operated from a desktop browser)
- **Responsive**: all views must work correctly on both form factors

## Access & Registration

| Feature | Detail |
|---------|--------|
| Guest mode | Users can place an order without creating an account. Only minimal data required (name, phone/email, address) |
| Email + password registration | Standard account creation with email verification |
| Google login | OAuth authentication with Google account |
| Password recovery | Standard recovery flow via email |

## Address Management

Registered users can manage multiple delivery addresses:

- Add, edit, and delete addresses
- Assign alias/label to each address (e.g., "Home", "Work")
- Add comments per address (e.g., "Floor 3, buzzer B", "Green gate")
- Select active address when placing an order
- Coverage validation: if the address is outside the delivery zone, the user is informed

## Catalog & Order Building

The catalog shows products available for delivery. Products marked "counter only" are hidden from online ordering.

| Element | Description |
|---------|-------------|
| Delivery formats | 1/2 kg, 1 kg, 2 kg, cup, container, etc. (admin-configurable) |
| Counter-only formats | Cone, small cup, etc. Hidden in delivery app |
| Flavor selection | Each format has a configurable flavor limit (3 or 4) |
| Out-of-stock flavors | Flavors disabled by admin are not shown |
| Extras | Extra-cost add-ons (toppings, sauce, mix-ins). Price per extra |
| Item comment | Free-text note per cart item |
| Order comment | General observations for the entire order |

## Delivery Costs

Shipping cost is calculated automatically based on distance (km) between the store and delivery address. Architecture must support future evolution to polygon-based zones.

- Cost shown to customer before confirming the order
- Zones and rates are admin-configurable
- If address is out of coverage, order cannot be completed as delivery

## Loyalty Program — Points

| Parameter | Definition |
|-----------|------------|
| Accumulation rate | 1 peso = 1 point (on order total, excluding shipping) |
| Point expiration | Points expire 3 months after accrual date |
| Redemption | Customer can redeem accumulated points as a discount on the order total |
| Discount coupons | Customer can enter a coupon code. Automatic validation before confirming payment |
| Counter validation | Admin can look up and credit points to a customer by phone or email |

## Payment Methods

- Mercado Pago (official API integration — fully online payment)
- Bank transfer
- Credit/debit card (integrated gateway)
- Cash on delivery (payment recorded as "pending" until admin confirms collection)

## User Panel

Registered users have a personal panel with access to:

- Real-time status of active order
- Order history
- Accumulated points balance and expiration dates
- Saved address management
- Account details (name, email, password)

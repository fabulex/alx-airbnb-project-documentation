from pathlib import Path

# Define USER_STORIES.md content again
user_stories_md = """# 🏡 Airbnb Clone Backend — User Stories

---

## 🧍‍♂️ User Authentication & Management

**1. User Registration**  
> As a new user, I want to create an account using my email or social login (Google/Facebook) so that I can access the platform.

**2. User Login**  
> As a registered user, I want to securely log in using my credentials so that I can access my personal dashboard.

**3. JWT Authentication**  
> As a user, I want my session to be securely managed using tokens so that I don’t have to log in repeatedly.

**4. Role-Based Access Control**  
> As an admin, I want to manage permissions for users and hosts so that only authorized people can perform specific actions.

**5. Profile Management**  
> As a user, I want to update my personal information and profile photo so that my account reflects my current details.

---

## 🏠 Property Listings Management

**6. Create Property Listing**  
> As a host, I want to add a new property with title, description, location, price, and photos so that guests can view and book it.

**7. Edit Property Listing**  
> As a host, I want to update property details when changes occur so that my listings remain accurate.

**8. Delete Property Listing**  
> As a host, I want to remove a property from my listings so that unavailable properties are not displayed.

**9. Image Upload**  
> As a host, I want to upload and manage images for my listings so that guests can see what the property looks like.

---

## 🔎 Search & Filtering

**10. Property Search**  
> As a guest, I want to search for properties by location, price, date, and amenities so that I can find suitable stays.

**11. Filtering & Pagination**  
> As a guest, I want to filter results and navigate pages efficiently so that I can browse listings easily.

---

## 📅 Booking Management

**12. Create Booking**  
> As a guest, I want to book a property for specific dates and ensure it’s available so that I can secure my stay.

**13. Cancel Booking**  
> As a guest, I want to cancel my booking according to policy so that I can manage changes in my plans.

**14. Booking Status Tracking**  
> As a guest, I want to see the status of my booking (Pending, Confirmed, Completed, or Canceled) so that I know my reservation’s state.

---

## 💳 Payments Integration

**15. Secure Payment**  
> As a guest, I want to pay for my booking securely using Stripe or PayPal so that my financial information is protected.

**16. Host Payouts**  
> As a host, I want to receive payment after a guest’s stay is completed so that I get paid for my property.

**17. Refund Handling**  
> As a guest, I want to receive refunds automatically when I cancel within policy so that I’m reimbursed fairly.

---

## ⭐ Reviews & Ratings

**18. Guest Reviews**  
> As a guest, I want to leave a review and rating for a property after my stay so that others can benefit from my feedback.

**19. Host Responses**  
> As a host, I want to respond to guest reviews so that I can maintain a good reputation and clarify feedback.

---

## ⚙️ Admin & Monitoring

**20. Admin Dashboard**  
> As an admin, I want to view analytics, manage users, listings, and bookings so that I can ensure platform health and compliance.
"""

# Save file
output_path = Path("/mnt/data/USER_STORIES.md")
output_path.write_text(user_stories_md, encoding="utf-8")

output_path

### **1️⃣ Comment & Document Code**

☐ Add **comments** for complex logic in Next.js, .NET, and SQL queries  
☐ Ensure **function headers** describe input, output, and purpose  
☐ Write/update **README** with setup instructions and API documentation  
☐ Clean up unnecessary or redundant **console logs** and debugging statements\

---

### **2️⃣ Clean Up API Calls**

☐ Ensure **consistent API response structure** (standardize success/error messages)  
☐ Remove **unused endpoints** or **deprecated API calls**  
☐ Validate **error handling** for all API routes (**try/catch, proper status codes**)  
☐ Optimize **database queries** to reduce redundant requests (e.g., use joins instead of multiple calls)  
☐ Implement **pagination** where necessary to prevent large data loads

---

### **3️⃣ Final Database Updates**

☐ Add **User SID (Security Identifier) field** to the user table

- Modify schema to store AD User SIDs
- Backfill existing users with their SIDs from AD  
    ☐ Ensure **indexes** are properly set for faster queries  
    ☐ Run final **database migrations** and confirm compatibility with existing data  
    ☐ Implement **referential integrity checks** (e.g., ensure foreign keys are correct)

---

### **4️⃣ Batch Job: Reconcile AD with Authoritative Database**

☐ Create a **batch job** to run at scheduled intervals to:

- Fetch all **Active Directory users**
- Compare with users in the **HR database**
- Identify **new users** (add them to DB if needed)
- Identify **inactive users** (flag for removal if no longer in AD)
- Update **user information** (sync AD data with DB if mismatched)  
    ☐ Log **all changes** for auditing (added, removed, updated users)  
    ☐ Implement **alerts or reports** for HR when changes are made

---

### **5️⃣ Authentication & Security**

☐ Ensure **JWT authentication** is properly implemented in .NET API  
☐ Restrict **API access based on roles** (HR admin, manager, etc.)  
☐ Implement **Active Directory login integration** for SSO (Single Sign-On)  
☐ Secure **environment variables** (store secrets properly in `.env` or secrets manager)  
☐ Enforce **HTTPS** and **secure cookies** in Next.js frontend  
☐ Conduct **security review** to check for vulnerabilities (SQL injection, CSRF, etc.)

---

# **🎯 Final Steps for UAT Readiness**

### **📌 After Completing Dev Tasks**

✅ **Run full integration tests** with all components  
✅ Deploy **UAT version** and verify logs  
✅ Have **test users** sign in and perform core actions  
✅ Collect **final feedback** before production

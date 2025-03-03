# Rate Limiting for GKE with Google Cloud Armor

## 💡 What is Google Cloud Armor?
**Google Cloud Armor** is a **DDoS protection** and **web application firewall (WAF)** that helps protect applications running on **Google Kubernetes Engine (GKE)**. It allows you to create **security policies** that can **filter incoming traffic** to your **GKE services**, helping prevent threats like **SQL injection**, **Cross-Site Scripting (XSS)**, **bot attacks**, and **DDoS attacks**.

---

## ⚙️ How Does GKE Work with Google Cloud Armor?

### 1. GKE Load Balancer Integration
When you expose a GKE service using a **LoadBalancer**, it creates a **Google Cloud HTTP(S) Load Balancer**. This load balancer is where you can attach a **Google Cloud Armor policy**.

### 2. Traffic Flow:
1. **User Request:** Incoming traffic hits the **External HTTP(S) Load Balancer**.
2. **Cloud Armor Policy Evaluation:** Before the request reaches GKE, the load balancer evaluates the traffic against the **Cloud Armor rules**.
3. **Allow or Block Traffic:** Depending on the rules, traffic is either **allowed** or **blocked**.
4. **Forward to GKE:** Valid requests are forwarded to the **GKE Ingress** or directly to **services/pods**.

### 3. Protection Scenarios:
- **Web Application Firewall (WAF):** Apply **preconfigured WAF rules** to block common vulnerabilities.
- **IP Allow/Deny Lists:** Restrict access based on **IP addresses**.
- **Rate Limiting:** Protect against **application-layer DDoS** by limiting requests.
- **Geo-Blocking:** Block traffic from **specific regions**.

---

## 🔨 How to Set Up Google Cloud Armor with GKE

### Step 1: Create a Security Policy
```bash
gcloud compute security-policies create report-service-rate-limit-policy \
    --description "Rate limiting for Report Service"

# Add a rate-limiting rule to allow max 100 requests per minute
gcloud compute security-policies rules create 1000 \
    --security-policy=report-service-rate-limit-policy \
    --expression="true" \
    --action="throttle" \
    --rate-limit-threshold-count=100 \
    --rate-limit-threshold-interval=60s \
    --conform-action="allow" \
    --exceed-action="deny-429"
```

### Step 2: Update the Knative Service Configuration
Add the `run.googleapis.com/security-policy` annotation to your **Knative Service** YAML file:

```yaml
metadata:
  annotations:
    run.googleapis.com/security-policy: report-service-rate-limit-policy
```

### Step 3: Deploy the Updated Configuration
```bash
kubectl apply -f report-service-v2-5-espv.yaml
```

### Step 4: Verify the Security Policy Application
```bash
gcloud compute security-policies describe report-service-rate-limit-policy
```

### Step 5: Test the Rate Limiting
```bash
for i in {1..120}; do curl -I https://report-service-v2-5-espv-27ahohymea-nw.a.run.app/; done
```

You should see **HTTP 429 (Too Many Requests)** after exceeding the **100 requests per minute** limit.

---

## 📈 Monitoring and Alerts
- Enable **Cloud Monitoring** and **Cloud Logging**.
- Set up **alerts** if your service is being throttled frequently, indicating potential abuse or misconfiguration.

---

## 💡 Additional Tips
- **Adjust Rate Limits** based on **service usage patterns**.
- Use **Cloud Armor WAF rules** for **additional security**.
- Configure **burst capacity** if needed:

```bash
--rate-limit-threshold-count=200 \
--rate-limit-threshold-interval=60s \
--conform-action="allow" \
--exceed-action="throttle"
```

---

Would you like help with **testing and validating** the rate-limiting behavior or need **advanced configurations** for different request types? 😊


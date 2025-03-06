# Setting Up Google Cloud Armor with Cloud Run via Load Balancer

This guide outlines the steps to protect your Cloud Run service using Google Cloud Armor by configuring an external HTTP(S) Load Balancer.

## Prerequisites

- **Google Cloud Project**: Ensure you have a project set up in the [Google Cloud Console](https://console.cloud.google.com/).
- **Cloud Run Service**: Deploy your application on Cloud Run. Refer to the [Cloud Run Quickstart Guide](https://cloud.google.com/run/docs/quickstarts) if needed.
- **gcloud CLI**: Install and configure the [Google Cloud SDK](https://cloud.google.com/sdk/docs/install).

## Steps

1. **Create a Serverless Network Endpoint Group (NEG):**

   A Serverless NEG allows the Load Balancer to route traffic to your Cloud Run service.

   ```bash
   gcloud compute network-endpoint-groups create my-serverless-neg \
       --region=us-central1 \
       --network-endpoint-type=serverless \
       --cloud-run-service=YOUR_CLOUD_RUN_SERVICE_NAME
   ```

   Replace `YOUR_CLOUD_RUN_SERVICE_NAME` with the name of your Cloud Run service.

2. **Set Up a Backend Service:**

   The Backend Service manages traffic to your Serverless NEG.

   ```bash
   gcloud compute backend-services create my-backend-service \
       --protocol=HTTP \
       --port-name=http \
       --global
   ```

   Add the NEG to the Backend Service:

   ```bash
   gcloud compute backend-services add-backend my-backend-service \
       --global \
       --network-endpoint-group=my-serverless-neg \
       --network-endpoint-group-region=us-central1
   ```

3. **Create a URL Map:**

   The URL Map defines how requests are routed to Backend Services based on the URL.

   ```bash
   gcloud compute url-maps create my-url-map \
       --default-service=my-backend-service
   ```

4. **Create an HTTP(S) Proxy:**

   The HTTP(S) Proxy processes incoming HTTP(S) requests and forwards them according to the URL Map.

   ```bash
   gcloud compute target-http-proxies create my-http-proxy \
       --url-map=my-url-map
   ```

5. **Create a Global Forwarding Rule:**

   The Forwarding Rule routes incoming requests to the HTTP(S) Proxy.

   ```bash
   gcloud compute forwarding-rules create my-forwarding-rule \
       --global \
       --target-http-proxy=my-http-proxy \
       --ports=80
   ```

6. **Create a Cloud Armor Security Policy:**

   Define security rules to protect your service.

   ```bash
   gcloud compute security-policies create my-security-policy \
       --description="Security policy for Cloud Run service"
   ```

   Add rules to the policy as needed, such as IP allowlists or deny lists.

7. **Apply the Security Policy to the Backend Service:**

   Associate the Cloud Armor policy with your Backend Service.

   ```bash
   gcloud compute backend-services update my-backend-service \
       --security-policy=my-security-policy \
       --global
   ```

8. **Update Cloud Run Ingress Settings:**

   Restrict direct access to your Cloud Run service to only allow traffic from the Load Balancer.

   ```bash
   gcloud run services update YOUR_CLOUD_RUN_SERVICE_NAME \
       --ingress internal-and-cloud-load-balancing
   ```

   Replace `YOUR_CLOUD_RUN_SERVICE_NAME` with your Cloud Run service name.

9. **Test the Setup:**

   Obtain the external IP address of your Load Balancer:

   ```bash
   gcloud compute forwarding-rules list
   ```

   Access your service via the Load Balancer's IP to ensure it's functioning correctly and that the Cloud Armor policies are enforced.

For a comprehensive walkthrough, refer to Google's official documentation on [setting up an HTTPS Load Balancer with serverless backends](https://cloud.google.com/load-balancing/docs/https/setting-up-https-serverless).

This setup ensures that all traffic to your Cloud Run service passes through the Load Balancer, allowing Cloud Armor to effectively protect your application.


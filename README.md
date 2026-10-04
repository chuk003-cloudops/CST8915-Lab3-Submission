# CST8915 Lab 3: Algonquin Pet Store on Azure

## Deployment status

The Python product service and Node.js order service are deployed on Azure App Service. RabbitMQ runs on a dedicated Azure VM. Live API testing returned `200 Order received`, and the broker queue increased from 4 to 5 messages. A subsequent order placed through the deployed Vue storefront succeeded and increased the queue to 6 messages.

The Azure for Students subscription's allowed deployment regions do not overlap the five regions supported by Azure Static Web Apps. A working App Service storefront backup uses the same Vue production build. This backup is an explicit platform deviation from the lab instructions. Exact compliance still requires deploying the frontend to Azure Static Web Apps using a supported subscription, or an instructor-approved exception.

## Demo video

Pending the final frontend platform decision. The video will show the real deployed browser application, backend environment-setting names with secret values hidden, GitHub Actions environment variables, an order, and RabbitMQ queue activity. Maximum length: five minutes.

## Deployed endpoints

- Product API: https://chuk8915product.azurewebsites.net/products
- Order API: https://chuk8915order.azurewebsites.net/orders
- Vue storefront backup: https://chuk8915storefront.azurewebsites.net/
- Azure Static Web Apps URL: pending supported subscription deployment.

## Service repositories from Lab 2

- [Order service](https://github.com/chuk003-cloudops/order-service)
- [Product service](https://github.com/chuk003-cloudops/product-service)
- [Storefront](https://github.com/chuk003-cloudops/store-front)

## Reflection questions

### 1. What challenges did you encounter when configuring environment variables in the GitHub Actions workflow?

Vue reads its API URLs during the build, so changing a server setting after deployment does not update the compiled frontend. The workflow now declares `VUE_APP_ORDER_SERVICE_URL` and `VUE_APP_PRODUCT_SERVICE_URL` at the job level, before `npm run build`. The other challenge was separating those public URLs from the private deployment token. The URLs belong in the workflow, while the token belongs in GitHub Secrets. The workflow produces a build artifact even while Azure's region policy prevents creating the required Static Web App.

### 2. How does deploying microservices on Azure Web App Service differ from running them locally?

Locally, each process can use localhost, a local `.env` file, and a manually chosen port. Azure App Service supplies the runtime and listening port, and the services use public HTTPS endpoints and application environment settings. RabbitMQ also needs a reachable address and a network rule for the App Service's outbound addresses. The live test initially timed out because the worker used an address from Azure's documented possible outbound set; updating the broker rule resolved the connection. Managed hosting removes much of the VM setup, but configuration and network access still have to be verified in the deployed environment.

### 3. Why is it important to use environment variables for configurations in a cloud environment?

Environment variables let the same source code run locally and in Azure with different endpoints, ports, and credentials. RabbitMQ is treated as an attached backing service through `RABBITMQ_CONNECTION_STRING`, so its address can change without rewriting the order-service code. Keeping that credential in Azure settings also keeps it out of the public repositories and the frontend bundle. The frontend's public service URLs are supplied at build time, while backend credentials remain runtime settings.

## Validation and setup notes

- Product service was rewritten in Python with four local tests passing.
- Order service has 17 behavioral tests, run with `node --test`. Its `package.json` deployment script matches the instructor's Lab 3 instructions.
- The Vue production build succeeded with the Azure backend URLs embedded. Webpack reported an image-size warning for the existing Algonquin background image.
- The GitHub Actions workflow builds Vue and retains the `dist` artifact. Its Static Web Apps deployment step runs when the deployment token is available.
- RabbitMQ's management interface is viewed through an SSH tunnel. The demo does not require a public management-port rule.
- No credential values are included in these repositories.

This repository is prepared work in progress and has not yet been submitted as a complete Lab 3 deliverable.

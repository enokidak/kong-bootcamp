# What's this?
This repository is a template for APIOps in Konnect.

When the API Spec is updated, after validating the API Spec, the Service and Route are created in Konnect.
At that time, a Global Plugin is automatically added to enforce the organization's governance.
After creation, a test script is executed, and if no issues are found, the Spec documentation associated with the API Product's version is updated.

When the API Product documentation is updated, only the API Product documentation is uploaded.

It provides the following flow.

```mermaid
graph TD;
    A[ git push ] -->| Modify API Spec | B[ Linting ]
    A -->| Modify API Doc | C[ Upload Doc to Konnect ]
    B --> D[ Convert OAS to deck ]
    D --> E[ Add Global Plugin ]
    E --> F[ Back Up Current Settings ]
    F --> G[ Diff Current Settings and kong.yaml ]
    G --> H[ Deploy ]
    H --> I[ Test ]
    I --> J[ Upload API Spec to Konnect ]
```



# Prerequisite and Limitation
The API Product documentation and OpenAPI Spec should be located as follows.
```sh
.
├── docs
│   ├── openapi
│   │   └── api-spec.yaml
│   └── product.md
```

The following in Konnect should be prepared in advance.
- Control Plane
- Dev Portal
- API Product
- API Product's Version

And you need to activate GitHub Runner.

# How to use
## 1. Set Repository Secret and Environment variables
```sh
# Konnect Token
gh secret set KONNECT_TOKEN --body "spat_7f4IKi5g...."
```

```sh
# Konnect Region
gh variable set KONNECT_REGION -b "us"
# Control Plane
gh variable set CONTROL_PLANE -b "default"
# Dev Portal Name
gh variable set PORTAL_NAME -b "default-dev-portal"
# API Product Name
gh variable set PRODUCT_NAME -b "httpbin"
# Version Name
gh variable set VERSION_NAME -b "v1"
# Tag to identify it from other entities.
gh variable set TAG -b "httpbin"
```

## 2. Build Container Image
If you want to create and manage your own image instead of the one I created (`imuratashared/konnect-apiops`), you can build and push it.

```sh
DOCKER_REPOSITORY=your_repository
docker build -t $DOCKER_REPOSITORY --platform linux/x86_64 .
docker push $DOCKER_REPOSITORY
```

## 3. Modify container image name in workflows dir.

After building image, modify `jobs.<job name>.container.image` to `<your_repository>`.
```sh
sed -i "s|imuratashared/konnect-apiops|$DOCKER_REPOSITORY|g" .github/workflows/*
```

## 4. Run Workflow
If you update `api-spec.yaml`, GitHub Runner runs `deploy_oas.yaml` and `upload_spec.yaml`.
After validating the Spec, the Service or Route is created and the API Spec file is uploaded.
If you update `product.md`, GitHub Runner runs `upload_doc.yaml`.
`product.md` is uploaded.

graph TD
    subgraph "Data Acquisition Layer"
        API[The Odds API / Sports Data APIs]
        S3_RAW[(AWS S3: Raw Datasets)]
        API -->|Fetch Odds/Stats| S3_RAW
    end

    subgraph "Processing & MLOps Pipeline"
        CELERY[Celery Worker: Movement Utils]
        REDIS[Redis: Task Queue]
        S3_MODELS[(AWS S3: 20+ Trained Models)]
        
        CELERY -->|Incremental Load| S3_RAW
        CELERY -->|Feature Engineering 1k+| FE[Feature Selection: SHAP/VIF]
        FE -->|Inference| ENSEMBLE{Ensemble Engine}
        
        S3_MODELS -->|CatBoost / TabNet / LSTM| ENSEMBLE
    end

    subgraph "Backend API Layer (Django REST Framework)"
        DB[(PostgreSQL)]
        DJANGO[Django REST Framework]
        STRIPE[Stripe API: Subscriptions]
        
        ENSEMBLE -->|Predictions| DB
        DJANGO -->|Query| DB
        DJANGO <-->|Webhooks| STRIPE
    end

    subgraph "Consumer Layer"
        REACT[React Frontend]
        MOBILE[Mobile / JWT Auth]
        DJANGO -->|JSON API Endpoints| REACT
        DJANGO -->|Secure Auth| MOBILE
    end

    style ENSEMBLE fill:#f96,stroke:#333,stroke-width:2px
    style DJANGO fill:#092,color:#fff

# 23 - Machine Learning en AWS

## Amazon SageMaker

Platform completa de ML.

### Capabilities

**Build**:
- Jupyter notebooks
- Built-in algorithms
- Bring your own algorithms
- Feature Store

**Train**:
- Managed training
- Distributed training
- Hyperparameter tuning
- Model debugging

**Deploy**:
- Real-time endpoints
- Batch transform
- Model monitoring
- A/B testing

### SageMaker Studio

IDE completo para ML.

```
Features:
- Jupyter notebooks
- Visual interface
- Git integration
- Experiment tracking
- Model registry
```

### Example: Train Model

```python
import sagemaker
from sagemaker import get_execution_role

role = get_execution_role()
sess = sagemaker.Session()

# Built-in XGBoost algorithm
xgboost = sagemaker.estimator.Estimator(
    image_uri=sagemaker.image_uris.retrieve("xgboost", region),
    role=role,
    instance_count=1,
    instance_type='ml.m5.xlarge',
    output_path=f's3://{bucket}/output'
)

# Set hyperparameters
xgboost.set_hyperparameters(
    objective='binary:logistic',
    num_round=100,
    max_depth=5
)

# Train
xgboost.fit({'train': 's3://bucket/train.csv'})

# Deploy
predictor = xgboost.deploy(
    initial_instance_count=1,
    instance_type='ml.t2.medium'
)

# Predict
result = predictor.predict(data)
```

### SageMaker Features

**Autopilot**:
```
AutoML
Upload data → Get best model
No code required
```

**Ground Truth**:
```
Data labeling service
Human labelers
Active learning
```

**Feature Store**:
```
Centralized feature repository
Online store (low-latency)
Offline store (training)
```

**Model Monitor**:
```
Detect drift
Data quality monitoring
Model quality monitoring
Alerts
```

## Amazon Rekognition

Image and video analysis.

### Capabilities

```
Object detection
Scene detection
Face detection and analysis
Face comparison
Face search
Celebrity recognition
Text in images (OCR)
Content moderation
Custom labels
```

### Use Cases

```
✓ Content moderation (detect unsafe content)
✓ Face verification (security)
✓ Text extraction (OCR documents)
✓ Celebrity detection
✓ Object/scene detection
```

### Example: Detect Labels

```python
import boto3

rekognition = boto3.client('rekognition')

response = rekognition.detect_labels(
    Image={'S3Object': {
        'Bucket': 'my-bucket',
        'Name': 'photo.jpg'
    }},
    MaxLabels=10,
    MinConfidence=75
)

for label in response['Labels']:
    print(f"{label['Name']}: {label['Confidence']:.2f}%")
```

### Rekognition para Odoo

```python
# Moderate user-uploaded images
def moderate_image(attachment_id):
    s3_key = get_s3_key(attachment_id)
    
    response = rekognition.detect_moderation_labels(
        Image={'S3Object': {
            'Bucket': 'odoo-attachments',
            'Name': s3_key
        }}
    )
    
    for label in response['ModerationLabels']:
        if label['Confidence'] > 80:
            # Flag attachment
            flag_attachment(attachment_id, label['Name'])
```

## Amazon Transcribe

Speech to text.

### Features
```
Automatic speech recognition (ASR)
Multiple languages
Custom vocabulary
Speaker identification
Channel identification
Real-time transcription
Batch transcription
```

### Example

```python
import boto3

transcribe = boto3.client('transcribe')

transcribe.start_transcription_job(
    TranscriptionJobName='my-job',
    Media={'MediaFileUri': 's3://bucket/audio.mp3'},
    MediaFormat='mp3',
    LanguageCode='en-US'
)

# Get results
response = transcribe.get_transcription_job(
    TranscriptionJobName='my-job'
)

transcript_uri = response['TranscriptionJob']['Transcript']['TranscriptFileUri']
```

## Amazon Polly

Text to speech.

### Features
```
Lifelike voices
Multiple languages
Neural voices
SSML support
Lexicons (pronunciation)
Speech marks
```

### Example

```python
import boto3

polly = boto3.client('polly')

response = polly.synthesize_speech(
    Text='Hello, this is Amazon Polly.',
    OutputFormat='mp3',
    VoiceId='Joanna',
    Engine='neural'
)

with open('speech.mp3', 'wb') as file:
    file.write(response['AudioStream'].read())
```

## Amazon Translate

Neural machine translation.

### Features
```
75+ languages
Real-time translation
Batch translation
Custom terminology
Automatic language detection
```

### Example

```python
import boto3

translate = boto3.client('translate')

response = translate.translate_text(
    Text='Hello, how are you?',
    SourceLanguageCode='en',
    TargetLanguageCode='es'
)

print(response['TranslatedText'])
# Hola, ¿cómo estás?
```

## Amazon Comprehend

Natural language processing (NLP).

### Capabilities

```
Entity recognition
Sentiment analysis
Key phrase extraction
Language detection
Topic modeling
Custom classification
Custom entity recognition
```

### Example: Sentiment Analysis

```python
import boto3

comprehend = boto3.client('comprehend')

response = comprehend.detect_sentiment(
    Text='I love this product! It works great.',
    LanguageCode='en'
)

print(response['Sentiment'])  # POSITIVE
print(response['SentimentScore'])
# {'Positive': 0.98, 'Negative': 0.01, 'Neutral': 0.01, 'Mixed': 0.00}
```

### Comprehend para Odoo

```python
# Analyze customer feedback
def analyze_feedback(feedback_text):
    response = comprehend.detect_sentiment(
        Text=feedback_text,
        LanguageCode='en'
    )
    
    sentiment = response['Sentiment']
    score = response['SentimentScore'][sentiment]
    
    # Update customer record
    if sentiment == 'NEGATIVE' and score > 0.8:
        create_support_ticket(feedback_text)
```

## Amazon Lex

Build conversational interfaces (chatbots).

### Features
```
Speech and text
Natural language understanding
Multi-turn conversations
Integration con Lambda
Pre-built connectors
```

### Use Case: Customer Service Bot

```
User: "I want to check my order status"
Lex: Intent = CheckOrder
Lex → Lambda (get order status)
Lambda → Odoo API
Response: "Your order #1234 shipped yesterday"
```

## Amazon Forecast

Time series forecasting.

### Use Cases
```
✓ Demand forecasting
✓ Inventory planning
✓ Resource planning
✓ Financial planning
```

### Example

```python
import boto3

forecast = boto3.client('forecast')

# Create dataset
forecast.create_dataset(
    DatasetName='sales_data',
    Domain='RETAIL',
    DatasetType='TARGET_TIME_SERIES',
    Schema={...}
)

# Import data
forecast.create_dataset_import_job(...)

# Train predictor
forecast.create_predictor(
    PredictorName='sales_predictor',
    ForecastHorizon=30  # 30 days ahead
)

# Generate forecast
forecast.create_forecast(...)
```

## Amazon Personalize

Personalization and recommendations.

### Use Cases
```
✓ Product recommendations
✓ Personalized search
✓ Targeted marketing
✓ Similar items
```

### Example: E-commerce

```
User behavior data → Personalize
↓
Recommendations:
- "Customers who bought X also bought Y"
- "Recommended for you"
- "Similar items"
```

## Amazon Textract

Extract text and data from documents.

### Capabilities
```
OCR (text extraction)
Forms extraction (key-value pairs)
Tables extraction
Handwriting recognition
Identity documents
Invoices and receipts
```

### Example: Invoice Processing

```python
import boto3

textract = boto3.client('textract')

response = textract.analyze_expense(
    Document={'S3Object': {
        'Bucket': 'invoices',
        'Name': 'invoice.pdf'
    }}
)

for expense in response['ExpenseDocuments']:
    for field in expense['SummaryFields']:
        print(f"{field['Type']['Text']}: {field['ValueDetection']['Text']}")
# Invoice Number: INV-001
# Total: $1,250.00
```

### Textract para Odoo

```python
# Auto-process vendor invoices
def process_invoice_upload(s3_key):
    response = textract.analyze_expense(
        Document={'S3Object': {'Bucket': 'uploads', 'Name': s3_key}}
    )
    
    invoice_data = extract_invoice_data(response)
    
    # Create draft invoice in Odoo
    odoo_api.create('account.move', {
        'partner_id': find_partner(invoice_data['vendor']),
        'invoice_date': invoice_data['date'],
        'invoice_line_ids': invoice_data['lines']
    })
```

## ML Architecture - Odoo Example

```
Scenario: Intelligent document processing

1. Upload
   User → Odoo → S3

2. Trigger
   S3 Event → Lambda

3. Process
   Lambda → Textract (extract data)
   Lambda → Comprehend (classify document)
   Lambda → Translate (if needed)

4. Store
   Lambda → Odoo API (create records)

5. Monitor
   SageMaker Model Monitor

Benefits:
✓ Automated data entry
✓ Reduced errors
✓ Faster processing
✓ Scalable
```

---

**Certificación**: Conocer servicios ML y sus use cases es importante.

**Siguiente**: [24 - Monitoring & Audit](24-monitoring.md)

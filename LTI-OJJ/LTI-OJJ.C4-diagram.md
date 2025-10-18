# LTI - Diagrama C4: AI Service (Servicio de Inteligencia Artificial)

## 📚 Modelo C4: Context, Containers, Components, Code

El modelo C4 proporciona una forma estructurada de visualizar la arquitectura de software en diferentes niveles de abstracción. Aplicaremos los 4 niveles al **AI Service**, el componente más crítico y complejo de LTI.

---

## 🌍 NIVEL 1: CONTEXT DIAGRAM (Diagrama de Contexto)

### Descripción
Vista de más alto nivel mostrando cómo el AI Service interactúa con usuarios y sistemas externos.

### Propósito del AI Service
El AI Service es el cerebro de LTI. Proporciona capacidades de inteligencia artificial y machine learning para:
- Parsing automático de CVs
- Matching semántico entre candidatos y vacantes
- Scoring predictivo de candidatos
- Detección de sesgos en procesos de selección
- Generación de contenido (descripciones de puestos)
- Análisis de feedback y sentimientos

### Actores y Sistemas

| Actor/Sistema | Relación con AI Service | Propósito |
|---------------|------------------------|-----------|
| **Reclutador** | Usuario indirecto | Beneficiario del scoring y recomendaciones |
| **Candidate Service** | Sistema cliente | Solicita parsing de CVs y scoring de aplicaciones |
| **Job Service** | Sistema cliente | Solicita mejoras en descripciones de puestos |
| **Analytics Service** | Sistema cliente | Solicita análisis predictivos y tendencias |
| **Training Data Store** | Sistema externo | Fuente de datos históricos para reentrenamiento |
| **Vector Database** | Sistema externo | Almacenamiento de embeddings para búsqueda semántica |
| **ML Model Registry** | Sistema externo | Repositorio de modelos entrenados y versionados |

---

## 📦 NIVEL 2: CONTAINER DIAGRAM (Diagrama de Contenedores)

### Descripción
Muestra los contenedores (aplicaciones, servicios, bases de datos) que componen el AI Service.

### Contenedores del AI Service

#### 1. **API Application (FastAPI)**
- **Tecnología**: Python 3.11 + FastAPI
- **Responsabilidad**: Exponee endpoints REST para funcionalidades de IA
- **Puerto**: 8080
- **Deployment**: Container Docker en Kubernetes
- **Características**:
  - Async/await para operaciones I/O
  - Auto-documentación (OpenAPI/Swagger)
  - Validación con Pydantic
  - Rate limiting por tenant

#### 2. **ML Processing Worker**
- **Tecnología**: Python + Celery
- **Responsabilidad**: Procesamiento asíncrono de tareas pesadas de ML
- **Deployment**: Multiple workers en Kubernetes
- **Queue**: RabbitMQ
- **Características**:
  - Procesa jobs en background (parsing CVs, reentrenamiento)
  - Escalado horizontal basado en queue depth
  - Retry logic con exponential backoff

#### 3. **Model Serving (TensorFlow Serving / TorchServe)**
- **Tecnología**: TensorFlow Serving 2.x / TorchServe
- **Responsabilidad**: Inferencia optimizada de modelos ML
- **Deployment**: Dedicated GPU pods
- **Características**:
  - Batch prediction para eficiencia
  - Model versioning y A/B testing
  - Hot-swapping de modelos sin downtime

#### 4. **Feature Store**
- **Tecnología**: Feast / Redis
- **Responsabilidad**: Almacenamiento y serving de features para modelos
- **Características**:
  - Features pre-computadas para baja latencia
  - Point-in-time correctness para training
  - Online/offline feature serving

#### 5. **PostgreSQL Database**
- **Responsabilidad**: Metadatos de modelos, training jobs, predictions
- **Schema**: `ai_service`
- **Tablas principales**:
  - `models` - Registro de modelos entrenados
  - `predictions` - Cache de predicciones recientes
  - `training_jobs` - Estado de jobs de entrenamiento
  - `feedback` - Feedback de usuarios para active learning

#### 6. **Vector Database (Pinecone/Weaviate)**
- **Responsabilidad**: Almacenamiento y búsqueda de embeddings
- **Uso**:
  - Embeddings de CVs (768-dim vectors)
  - Embeddings de job descriptions
  - Semantic search con cosine similarity

#### 7. **Object Storage (S3)**
- **Responsabilidad**: Almacenamiento de modelos entrenados, datasets
- **Estructura**:
  ```
  /models/{model_name}/{version}/
  /training-data/{date}/
  /parsed-cvs/{candidate_id}/
  ```

---

## 🔧 NIVEL 3: COMPONENT DIAGRAM (Diagrama de Componentes)

### Descripción
Detalla los componentes internos del AI Service API Application.

### Componentes de la API Application

#### **1. API Layer (Controllers/Routers)**

##### CV Parser Controller
- **Ruta**: `POST /api/v1/parse-cv`
- **Responsabilidad**: Extrae información estructurada de CVs
- **Input**: Archivo PDF/DOCX o texto plano
- **Output**: JSON con datos parseados (experiencia, educación, skills)
- **Dependencias**: CVParserService, DocumentPreprocessor

##### Scoring Controller
- **Ruta**: `POST /api/v1/score-candidate`
- **Responsabilidad**: Calcula fit score entre candidato y vacante
- **Input**: `{candidate_id, job_posting_id}`
- **Output**: `{score: 0-100, explanation: {...}, confidence: 0-1}`
- **Dependencias**: ScoringService, FeatureStore

##### Matching Controller
- **Ruta**: `POST /api/v1/match-candidates`
- **Responsabilidad**: Encuentra candidatos similares a una vacante
- **Input**: `{job_posting_id, limit: 20}`
- **Output**: Lista de candidatos rankeados con scores
- **Dependencias**: MatchingService, VectorDatabase

##### Job Description Generator Controller
- **Ruta**: `POST /api/v1/generate-job-description`
- **Responsabilidad**: Genera/mejora descripciones de puestos con IA
- **Input**: `{title, requirements[], keywords[]}`
- **Output**: Descripción optimizada y libre de sesgos
- **Dependencias**: GenerationService, BiasDetector

##### Bias Detection Controller
- **Ruta**: `POST /api/v1/detect-bias`
- **Responsabilidad**: Identifica sesgos en texto o proceso
- **Input**: Texto a analizar o ID de aplicación
- **Output**: Lista de sesgos detectados con severidad
- **Dependencias**: BiasDetectionService

#### **2. Service Layer (Business Logic)**

##### CVParserService
```python
class CVParserService:
    def parse_cv(self, file_content: bytes, file_type: str) -> ParsedCV:
        """
        Pipeline de parsing:
        1. Conversión a texto (PDF/DOCX → texto)
        2. Limpieza y normalización
        3. NER (Named Entity Recognition) para extraer nombres, fechas, empresas
        4. Sección detection (experiencia, educación, skills)
        5. Parsing estructurado con regex + ML
        6. Validación y post-procesamiento
        """
        pass
    
    def extract_skills(self, text: str) -> List[Skill]:
        """Extrae skills usando NLP + diccionario de skills"""
        pass
    
    def extract_experience(self, text: str) -> List[Experience]:
        """Identifica bloques de experiencia laboral"""
        pass
```

**Dependencias**:
- `DocumentPreprocessor`: Limpieza de texto
- `NERModel`: Named Entity Recognition (spaCy)
- `SkillExtractor`: Extracción de habilidades técnicas
- `ExperienceParser`: Parsing de experiencia laboral

##### ScoringService
```python
class ScoringService:
    def calculate_score(self, candidate: Candidate, job: JobPosting) -> ScoringResult:
        """
        Algoritmo de scoring multi-factor:
        1. Skills match (40%) - Overlap de skills requeridas
        2. Experience match (30%) - Años y relevancia
        3. Education match (15%) - Nivel educativo
        4. Location/Remote fit (10%)
        5. Cultural fit indicators (5%)
        
        Output: Score 0-100 + explicación detallada
        """
        pass
    
    def predict_success_probability(self, candidate_id: str, job_id: str) -> float:
        """Predicción de éxito usando modelo ML entrenado"""
        pass
```

**Modelos ML utilizados**:
- `SkillMatchingModel`: BERT fine-tuned para matching de skills
- `SuccessPredictionModel`: XGBoost entrenado con datos históricos
- `CulturalFitModel`: Transformer para análisis de fit cultural

##### MatchingService
```python
class MatchingService:
    def find_similar_candidates(self, job_id: str, limit: int) -> List[Match]:
        """
        Semantic search en vector database:
        1. Obtener embedding de job description
        2. Búsqueda de k-nearest neighbors en vector DB
        3. Re-ranking con modelo de scoring
        4. Aplicar filtros duros (location, visa, salary)
        """
        pass
    
    def recommend_jobs_for_candidate(self, candidate_id: str) -> List[JobMatch]:
        """Recomienda vacantes para un candidato"""
        pass
```

**Dependencias**:
- `EmbeddingGenerator`: Genera embeddings con Sentence-BERT
- `VectorSearchEngine`: Interface con Pinecone/Weaviate
- `ReRankingModel`: Modelo de re-ranking para refinar resultados

##### BiasDetectionService
```python
class BiasDetectionService:
    def detect_text_bias(self, text: str) -> List[BiasDetection]:
        """
        Detecta sesgos en texto:
        - Lenguaje con sesgo de género
        - Referencias a edad
        - Requisitos discriminatorios
        - Lenguaje no inclusivo
        """
        pass
    
    def analyze_pipeline_bias(self, job_id: str) -> BiasAnalysis:
        """
        Analiza sesgos en proceso de selección:
        - Distribución demográfica por etapa
        - Tasa de avance por grupos
        - Identificación de bottlenecks discriminatorios
        """
        pass
```

**Modelos**:
- `GenderBiasDetector`: Clasifica lenguaje con sesgo de género
- `InclusivityScorer`: Puntúa inclusividad del texto
- `FairnessAnalyzer`: Analiza equidad estadística en proceso

##### GenerationService
```python
class GenerationService:
    def generate_job_description(self, params: JobParams) -> str:
        """
        Genera descripción usando GPT-4:
        1. Construir prompt con mejores prácticas
        2. Llamada a OpenAI API
        3. Post-procesamiento (bias removal)
        4. Validación de calidad
        """
        pass
    
    def improve_description(self, original: str) -> str:
        """Mejora descripción existente"""
        pass
```

**Dependencias**:
- `OpenAI API`: GPT-4 para generación
- `PromptTemplate`: Templates optimizados
- `QualityValidator`: Valida salida del modelo

#### **3. Data Access Layer**

##### ModelRepository
```python
class ModelRepository:
    def get_model(self, model_name: str, version: str) -> MLModel:
        """Carga modelo desde registry"""
        pass
    
    def save_prediction(self, prediction: Prediction) -> None:
        """Guarda predicción para auditoría"""
        pass
```

##### FeatureRepository
```python
class FeatureRepository:
    def get_candidate_features(self, candidate_id: str) -> Features:
        """Obtiene features pre-computadas de candidato"""
        pass
    
    def compute_features_on_demand(self, candidate: Candidate) -> Features:
        """Computa features en tiempo real si no existen"""
        pass
```

##### VectorRepository
```python
class VectorRepository:
    def upsert_embedding(self, entity_id: str, vector: List[float]) -> None:
        """Inserta/actualiza embedding en vector DB"""
        pass
    
    def search_similar(self, vector: List[float], top_k: int) -> List[Match]:
        """Búsqueda de similitud"""
        pass
```

#### **4. Infrastructure Layer**

##### DocumentPreprocessor
- Limpieza de texto (encoding, whitespace)
- Conversión de formatos (PDF/DOCX → texto)
- Tokenización y normalización

##### EmbeddingGenerator
- Genera embeddings usando Sentence-BERT
- Batch processing para eficiencia
- Caching de embeddings computados

##### ModelLoader
- Lazy loading de modelos ML
- Gestión de memoria (unload models inactivos)
- Model versioning

##### MetricsCollector
- Latencia de predicciones
- Distribución de scores
- Model drift detection

---

## 💻 NIVEL 4: CODE DIAGRAM (Diagrama de Código)

### Descripción
Detalle de implementación de un componente específico: **ScoringService**

### Implementación del ScoringService

```python
# ai_service/services/scoring_service.py

from typing import Dict, List, Tuple
from dataclasses import dataclass
import numpy as np
from sklearn.preprocessing import MinMaxScaler

from ai_service.models import Candidate, JobPosting
from ai_service.ml.models import SkillMatchingModel, SuccessPredictionModel
from ai_service.repositories import FeatureRepository, ModelRepository
from ai_service.utils.logger import logger


@dataclass
class ScoringResult:
    """Resultado del scoring de un candidato"""
    score: int  # 0-100
    confidence: float  # 0-1
    explanation: Dict[str, any]
    factors: Dict[str, float]
    

class ScoringService:
    """
    Servicio para calcular el fit score entre candidatos y vacantes.
    
    Utiliza múltiples factores ponderados:
    - Skills match (40%)
    - Experience match (30%)
    - Education match (15%)
    - Location/Remote compatibility (10%)
    - Cultural fit indicators (5%)
    """
    
    # Pesos de cada factor (deben sumar 1.0)
    WEIGHTS = {
        'skills': 0.40,
        'experience': 0.30,
        'education': 0.15,
        'location': 0.10,
        'culture': 0.05
    }
    
    def __init__(
        self,
        feature_repo: FeatureRepository,
        model_repo: ModelRepository,
        skill_model: SkillMatchingModel,
        success_model: SuccessPredictionModel
    ):
        self.feature_repo = feature_repo
        self.model_repo = model_repo
        self.skill_model = skill_model
        self.success_model = success_model
        self.scaler = MinMaxScaler()
        
    def calculate_score(
        self, 
        candidate: Candidate, 
        job: JobPosting
    ) -> ScoringResult:
        """
        Calcula score multi-factor entre candidato y vacante.
        
        Args:
            candidate: Candidato a evaluar
            job: Vacante objetivo
            
        Returns:
            ScoringResult con score, explicación y desglose de factores
        """
        logger.info(f"Calculating score for candidate {candidate.id} and job {job.id}")
        
        # Calcular cada factor individual
        skills_score = self._calculate_skills_match(candidate, job)
        experience_score = self._calculate_experience_match(candidate, job)
        education_score = self._calculate_education_match(candidate, job)
        location_score = self._calculate_location_match(candidate, job)
        culture_score = self._calculate_culture_fit(candidate, job)
        
        # Agregar scores con pesos
        weighted_score = (
            skills_score * self.WEIGHTS['skills'] +
            experience_score * self.WEIGHTS['experience'] +
            education_score * self.WEIGHTS['education'] +
            location_score * self.WEIGHTS['location'] +
            culture_score * self.WEIGHTS['culture']
        )
        
        # Normalizar a escala 0-100
        final_score = int(weighted_score * 100)
        
        # Calcular confianza basada en data quality
        confidence = self._calculate_confidence(candidate, job)
        
        # Construir explicación detallada
        explanation = self._build_explanation(
            candidate, job, 
            skills_score, experience_score, 
            education_score, location_score, culture_score
        )
        
        factors = {
            'skills': round(skills_score, 2),
            'experience': round(experience_score, 2),
            'education': round(education_score, 2),
            'location': round(location_score, 2),
            'culture': round(culture_score, 2)
        }
        
        # Guardar predicción para auditoría
        self._save_prediction(candidate.id, job.id, final_score, factors)
        
        return ScoringResult(
            score=final_score,
            confidence=confidence,
            explanation=explanation,
            factors=factors
        )
    
    def _calculate_skills_match(
        self, 
        candidate: Candidate, 
        job: JobPosting
    ) -> float:
        """
        Calcula match de skills usando modelo de NLP.
        
        Utiliza:
        1. Exact match de skills requeridas
        2. Semantic similarity de skills relacionadas (usando embeddings)
        3. Peso por importancia (required vs nice-to-have)
        """
        required_skills = set(job.required_skills)
        nice_skills = set(job.nice_to_have_skills)
        candidate_skills = set(candidate.skills)
        
        # Exact matches
        required_matches = required_skills & candidate_skills
        nice_matches = nice_skills & candidate_skills
        
        # Semantic matches usando modelo ML
        semantic_score = self.skill_model.calculate_semantic_similarity(
            candidate.skills,
            job.required_skills + job.nice_to_have_skills
        )
        
        # Weighted score
        required_coverage = len(required_matches) / len(required_skills) if required_skills else 1.0
        nice_coverage = len(nice_matches) / len(nice_skills) if nice_skills else 0.0
        
        # 70% required, 15% nice-to-have, 15% semantic
        score = (
            0.70 * required_coverage +
            0.15 * nice_coverage +
            0.15 * semantic_score
        )
        
        return min(score, 1.0)  # Cap at 1.0
    
    def _calculate_experience_match(
        self, 
        candidate: Candidate, 
        job: JobPosting
    ) -> float:
        """
        Evalúa match de experiencia laboral.
        
        Considera:
        - Años de experiencia total
        - Experiencia relevante en el sector
        - Nivel de seniority
        - Progresión de carrera
        """
        required_years = job.required_years_experience
        candidate_years = candidate.years_experience
        
        # Base score por años
        if candidate_years >= required_years:
            years_score = 1.0
        else:
            years_score = candidate_years / required_years
        
        # Bonus por experiencia relevante en sector/rol similar
        relevant_exp = self._calculate_relevant_experience(candidate, job)
        relevance_bonus = relevant_exp * 0.2  # Up to 20% bonus
        
        # Seniority match
        seniority_match = self._match_seniority_level(candidate, job)
        
        # Combined score
        score = (
            0.50 * years_score +
            0.30 * seniority_match +
            0.20 * relevance_bonus
        )
        
        return min(score + relevance_bonus, 1.0)
    
    def _calculate_education_match(
        self, 
        candidate: Candidate, 
        job: JobPosting
    ) -> float:
        """
        Evalúa match de educación.
        
        Factores:
        - Nivel educativo (Bachelor, Master, PhD)
        - Campo de estudio relevante
        - Institución (ranking opcional)
        - Certificaciones adicionales
        """
        if not job.required_education:
            return 1.0  # No requirement = full score
        
        education_levels = {
            'high_school': 1,
            'associate': 2,
            'bachelor': 3,
            'master': 4,
            'phd': 5
        }
        
        required_level = education_levels.get(job.required_education, 0)
        candidate_level = education_levels.get(candidate.education_level, 0)
        
        # Base score
        if candidate_level >= required_level:
            level_score = 1.0
        else:
            level_score = candidate_level / required_level if required_level > 0 else 0.0
        
        # Field of study match
        field_match = self._match_education_field(candidate, job)
        
        # Certifications bonus
        cert_bonus = len(candidate.certifications) * 0.05  # 5% per cert, max 20%
        cert_bonus = min(cert_bonus, 0.2)
        
        score = 0.70 * level_score + 0.30 * field_match
        
        return min(score + cert_bonus, 1.0)
    
    def _calculate_location_match(
        self, 
        candidate: Candidate, 
        job: JobPosting
    ) -> float:
        """
        Evalúa compatibilidad de ubicación.
        
        Considera:
        - Remote vs On-site
        - Distancia geográfica
        - Willingness to relocate
        - Timezone overlap
        """
        if job.remote_policy == 'fully_remote':
            return 1.0  # Perfect match for remote
        
        if job.remote_policy == 'hybrid':
            if candidate.location_proximity(job.location) < 50:  # km
                return 1.0
            elif candidate.willing_to_relocate:
                return 0.7
            else:
                return 0.3
        
        if job.remote_policy == 'on_site':
            if candidate.location == job.location:
                return 1.0
            elif candidate.willing_to_relocate:
                return 0.8
            else:
                return 0.1  # Very poor match
        
        return 0.5  # Default
    
    def _calculate_culture_fit(
        self, 
        candidate: Candidate, 
        job: JobPosting
    ) -> float:
        """
        Estima fit cultural usando ML.
        
        Basado en:
        - Análisis de lenguaje en cover letter
        - Valores expresados en perfil
        - Historial de permanencia en empresas
        - Preferencias de trabajo (startup vs corporate)
        """
        # Extract features
        features = self.feature_repo.get_culture_features(candidate)
        
        # Predict culture fit (modelo entrenado con feedback histórico)
        culture_score = self.success_model.predict_culture_fit(
            candidate_features=features,
            company_culture=job.organization.culture_profile
        )
        
        return culture_score
    
    def _calculate_confidence(
        self, 
        candidate: Candidate, 
        job: JobPosting
    ) -> float:
        """
        Calcula confianza en el score basado en calidad de datos.
        
        Penaliza:
        - CVs incompletos
        - Datos faltantes
        - Perfiles muy antiguos
        """
        completeness_score = candidate.profile_completeness  # 0-1
        recency_score = candidate.profile_recency_score  # 0-1
        data_quality = job.description_quality_score  # 0-1
        
        confidence = (
            0.50 * completeness_score +
            0.30 * data_quality +
            0.20 * recency_score
        )
        
        return confidence
    
    def _build_explanation(self, candidate, job, *scores) -> Dict:
        """Construye explicación human-readable del scoring"""
        skills_score, exp_score, edu_score, loc_score, culture_score = scores
        
        explanation = {
            'summary': self._generate_summary(candidate, job, scores),
            'strengths': self._identify_strengths(scores),
            'gaps': self._identify_gaps(scores),
            'recommendations': self._generate_recommendations(candidate, job, scores)
        }
        
        return explanation
    
    def _save_prediction(self, candidate_id, job_id, score, factors):
        """Guarda predicción en DB para auditoría y reentrenamiento"""
        self.model_repo.save_prediction(
            candidate_id=candidate_id,
            job_id=job_id,
            score=score,
            factors=factors,
            model_version=self.skill_model.version,
            timestamp=datetime.utcnow()
        )
    
    # ... más métodos auxiliares ...


# Ejemplo de uso
if __name__ == "__main__":
    # Dependency injection
    feature_repo = FeatureRepository()
    model_repo = ModelRepository()
    skill_model = SkillMatchingModel.load("skill-matcher-v2.1")
    success_model = SuccessPredictionModel.load("success-predictor-v1.3")
    
    scoring_service = ScoringService(
        feature_repo, model_repo, skill_model, success_model
    )
    
    # Calcular score
    result = scoring_service.calculate_score(candidate, job)
    
    print(f"Score: {result.score}/100 (confidence: {result.confidence:.2f})")
    print(f"Factors: {result.factors}")
    print(f"Explanation: {result.explanation}")
```

### Estructura de Clases (UML simplificado)

```
┌─────────────────────────┐
│   ScoringService        │
├─────────────────────────┤
│ - feature_repo          │
│ - model_repo            │
│ - skill_model           │
│ - success_model         │
│ - WEIGHTS: Dict         │
├─────────────────────────┤
│ + calculate_score()     │
│ - _calculate_skills()   │
│ - _calculate_exp()      │
│ - _calculate_edu()      │
│ - _calculate_location() │
│ - _calculate_culture()  │
│ - _build_explanation()  │
└─────────────────────────┘
         │
         │ uses
         ▼
┌─────────────────────────┐
│  SkillMatchingModel     │
├─────────────────────────┤
│ + calculate_similarity()│
│ + predict_match()       │
└─────────────────────────┘

         │
         │ uses
         ▼
┌─────────────────────────┐
│  FeatureRepository      │
├─────────────────────────┤
│ + get_candidate_feat()  │
│ + get_culture_feat()    │
└─────────────────────────┘
```

---

## 📊 Métricas y Monitoring del AI Service

### SLIs (Service Level Indicators)

| Métrica | Objetivo | Alerta |
|---------|----------|--------|
| **CV Parsing Latency** | p95 < 3s | > 5s |
| **Scoring Latency** | p95 < 500ms | > 1s |
| **Model Accuracy** | > 85% | < 80% |
| **API Availability** | > 99.5% | < 99% |
| **Error Rate** | < 0.5% | > 1% |

### Dashboards

1. **Operational Dashboard**
   - Request rate y latencia
   - Error rates por endpoint
   - Resource utilization (CPU/Memory/GPU)

2. **ML Performance Dashboard**
   - Model accuracy trends
   - Prediction distribution
   - Feature importance
   - Model drift detection

3. **Business Metrics Dashboard**
   - Scoring distribution (avg score per job)
   - Top skills detected
   - Bias detection alerts
   - User satisfaction (feedback)

---

## 🔐 Seguridad y Compliance

### Data Protection
- CVs encriptados at-rest y in-transit
- PII (Personally Identifiable Information) tokenizada
- Retención de datos según política GDPR

### Model Security
- Models versionados y firmados digitalmente
- Input validation estricta (evitar adversarial attacks)
- Rate limiting por tenant
- Audit log de todas las predicciones

### Bias Mitigation
- Auditoría regular de modelos por fairness
- Monitoring de distribución demográfica en predicciones
- Alerts automáticas si se detecta discriminación estadística

---

## 🚀 Deployment y Scaling

### Kubernetes Deployment

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: ai-service-api
spec:
  replicas: 3
  selector:
    matchLabels:
      app: ai-service-api
  template:
    metadata:
      labels:
        app: ai-service-api
    spec:
      containers:
      - name: api
        image: lti/ai-service:v2.3.1
        resources:
          requests:
            memory: "2Gi"
            cpu: "1000m"
          limits:
            memory: "4Gi"
            cpu: "2000m"
        env:
        - name: MODEL_VERSION
          value: "v2.1"
        livenessProbe:
          httpGet:
            path: /health
            port: 8080
        readinessProbe:
          httpGet:
            path: /ready
            port: 8080
```

### Scaling Strategy
- **Horizontal**: Auto-scaling basado en CPU (target: 70%)
- **Vertical**: GPU pods para model serving (escalado manual)
- **Predictive**: Scale-up preventivo antes de horas pico

---

## 📈 Roadmap del AI Service

### Q1 2026
- [ ] Multi-language support (CVs en español, francés, alemán)
- [ ] Real-time model retraining pipeline
- [ ] Explainable AI (SHAP values para interpretabilidad)

### Q2 2026
- [ ] Computer vision para análisis de portfolios/GitHub
- [ ] Voice analysis en video interviews (sentiment, tone
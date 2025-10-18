graph TB
    subgraph CU1["<b>CASO DE USO #1: PUBLICACIÓN Y FILTRADO AUTOMATIZADO</b>"]
        A1[👤 Reclutador crea vacante] --> A2[✍️ Editor inteligente sugiere mejoras]
        A2 --> A3[🎯 Define requisitos y pesos]
        A3 --> A4[📢 Publicación multicanal 1-clic]
        A4 --> A5[📊 200 CVs recibidos en 48h]
        A5 --> A6[🤖 Parsing automático de CVs]
        A6 --> A7{🧠 IA: Scoring 0-100}
        A7 -->|Score > 75| A8[✅ Para Revisar: 15 candidatos]
        A7 -->|Score 50-75| A9[🤔 Tal vez: 45 candidatos]
        A7 -->|Score < 50| A10[❌ Rechazo automático cortés]
        A8 --> A11[🔔 Alerta a reclutador]
        A11 --> A12[👀 Revisión final 2 horas]
        A12 --> A13[🎯 15 candidatos → 13 a entrevista]
        
        style A1 fill:#e3f2fd,stroke:#1976d2,stroke-width:2px
        style A7 fill:#fff9c4,stroke:#f57f17,stroke-width:3px
        style A8 fill:#c8e6c9,stroke:#388e3c,stroke-width:2px
        style A10 fill:#ffcdd2,stroke:#c62828,stroke-width:2px
        style A13 fill:#4caf50,stroke:#2e7d32,stroke-width:3px,color:#fff
    end
    
    subgraph CU2["<b>CASO DE USO #2: COORDINACIÓN COLABORATIVA DE ENTREVISTAS</b>"]
        B1[📋 5 finalistas seleccionados] --> B2[⚙️ Reclutador configura workflow]
        B2 --> B3[📧 Invitaciones automáticas enviadas]
        B3 --> B4[📅 Candidatos seleccionan horarios]
        B4 --> B5[🤖 Sistema encuentra slots compatibles]
        B5 --> B6[✅ 15 entrevistas programadas]
        
        B6 --> B7[👔 RONDA 1: Screening Técnico]
        B7 --> B8[💻 Tech Lead evalúa en scorecard]
        B8 --> B9{Decisión}
        B9 -->|Avanza| B10[👔 RONDA 2: Técnica Profunda]
        B9 -->|No avanza| B11[❌ Feedback automático]
        
        B10 --> B12[💻 Tech Lead + Dev evalúan]
        B12 --> B13{Decisión}
        B13 -->|Avanza| B14[👔 RONDA 3: Cultural + Oferta]
        B13 -->|No avanza| B15[❌ Feedback constructivo]
        
        B14 --> B16[🤝 Manager + CTO evalúan]
        B16 --> B17[📊 Matriz comparativa 5 finalistas]
        B17 --> B18[🏆 2 candidatos reciben oferta]
        B18 --> B19[⏱️ Proceso completo: 1 semana vs 3 semanas]
        
        style B1 fill:#e3f2fd,stroke:#1976d2,stroke-width:2px
        style B5 fill:#fff9c4,stroke:#f57f17,stroke-width:2px
        style B9 fill:#fff9c4,stroke:#f57f17,stroke-width:3px
        style B13 fill:#fff9c4,stroke:#f57f17,stroke-width:3px
        style B17 fill:#ce93d8,stroke:#7b1fa2,stroke-width:2px
        style B19 fill:#4caf50,stroke:#2e7d32,stroke-width:3px,color:#fff
    end
    
    subgraph CU3["<b>CASO DE USO #3: ANÁLISIS Y OPTIMIZACIÓN</b>"]
        C1[📊 Head of Talent accede dashboard] --> C2[📈 Métricas clave 12 meses]
        C2 --> C3[🔍 Análisis por fuente reclutamiento]
        
        C3 --> C4{Insight: Referidos}
        C4 --> C5[💡 2.5x mejor conversión<br/>80% menor coste]
        
        C2 --> C6[🔍 Análisis funnel conversión]
        C6 --> C7{Bottleneck detectado}
        C7 --> C8[⚠️ Entre entrevista 1 y 2<br/>Demora 2 semanas]
        
        C2 --> C9[🔍 Análisis por posición]
        C9 --> C10{Problema: Tech}
        C10 --> C11[⏱️ 68 días promedio<br/>vs 28 días sales]
        
        C2 --> C12[🔍 Análisis DEI]
        C12 --> C13{Alerta: Sesgo}
        C13 --> C14[⚠️ 40% mujeres aplican<br/>25% contratadas]
        
        C5 --> C15[🤖 IA genera recomendaciones]
        C8 --> C15
        C11 --> C15
        C14 --> C15
        
        C15 --> C16[✅ Aumentar programa referidos +200%]
        C15 --> C17[✅ Reducir tiempo entre entrevistas 14→5 días]
        C15 --> C18[✅ Ajustar filtros automáticos]
        C15 --> C19[✅ Activar modo diversidad]
        C15 --> C20[✅ Redistribuir presupuesto canales]
        
        C16 --> C21[📊 Seguimiento continuo]
        C17 --> C21
        C18 --> C21
        C19 --> C21
        C20 --> C21
        
        C21 --> C22[💰 Resultados 6 meses después]
        C22 --> C23[✨ CPH: 4.200€ → 2.900€<br/>✨ TTH: 42 días → 28 días<br/>✨ Diversidad: 25% → 38%]
        
        style C1 fill:#e3f2fd,stroke:#1976d2,stroke-width:2px
        style C4 fill:#fff9c4,stroke:#f57f17,stroke-width:2px
        style C7 fill:#fff9c4,stroke:#f57f17,stroke-width:2px
        style C10 fill:#fff9c4,stroke:#f57f17,stroke-width:2px
        style C13 fill:#fff9c4,stroke:#f57f17,stroke-width:2px
        style C15 fill:#ce93d8,stroke:#7b1fa2,stroke-width:3px
        style C23 fill:#4caf50,stroke:#2e7d32,stroke-width:3px,color:#fff
    end
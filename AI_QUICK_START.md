# AI Integration Quick Start Guide

## 🚀 Quick Setup (5 minutes)

### Step 1: Install Dependencies

```bash
# For OpenAI (recommended for quick start)
pip install openai

# OR for Anthropic Claude
pip install anthropic

# Optional: For local ML models
pip install scikit-learn numpy pandas
```

### Step 2: Set API Key

```bash
# For OpenAI
export OPENAI_API_KEY="your-api-key-here"

# OR for Claude
export ANTHROPIC_API_KEY="your-api-key-here"
```

### Step 3: Integrate into Report Generation

Add to `src/ecg/ecg_report_generator.py`:

```python
# At the top of the file
from ai.report_enhancer import enhance_report_with_ai

# In generate_ecg_report function, after gathering metrics:
def generate_ecg_report(...):
    # ... existing code ...
    
    # Add AI enhancement
    try:
        ai_enhancement = enhance_report_with_ai(
            metrics={
                'HR': int(HR) if HR.isdigit() else 70,
                'PR': int(PR) if PR.isdigit() else 160,
                'QRS': int(QRS) if QRS.isdigit() else 90,
                'QTc': int(QTc) if QTc.isdigit() else 430,
                'ST': int(ST) if ST.isdigit() else 0,
                'QRS_Axis': int(QRS_Axis) if QRS_Axis.isdigit() else 0
            },
            arrhythmias=arrhythmias_list,  # List of detected arrhythmias
            patient_data={
                'age': patient.get('age', 'N/A') if patient else 'N/A',
                'gender': patient.get('gender', 'N/A') if patient else 'N/A'
            }
        )
        
        # Add AI summary to report
        story.append(Spacer(1, 12))
        story.append(Paragraph("<b>AI-Powered Executive Summary</b>", styles['Heading2']))
        story.append(Paragraph(ai_enhancement['executive_summary'], styles['Normal']))
        
        # Add intelligent findings
        if ai_enhancement['intelligent_findings']:
            story.append(Spacer(1, 12))
            story.append(Paragraph("<b>Intelligent Findings</b>", styles['Heading2']))
            for finding in ai_enhancement['intelligent_findings']:
                story.append(Paragraph(f"<b>{finding['finding']}</b>", styles['Heading3']))
                story.append(Paragraph(finding['explanation'], styles['Normal']))
                story.append(Paragraph(f"Recommendations: {', '.join(finding['recommendations'])}", styles['Normal']))
        
        # Add risk assessment
        risk = ai_enhancement['risk_assessment']
        story.append(Spacer(1, 12))
        story.append(Paragraph(f"<b>Risk Assessment</b>", styles['Heading2']))
        story.append(Paragraph(f"Overall Risk Score: {risk['risk_score']}/100 ({risk['risk_level']} Risk)", styles['Normal']))
        if risk['recommendations']:
            story.append(Paragraph("Recommendations:", styles['Normal']))
            for rec in risk['recommendations']:
                story.append(Paragraph(f"• {rec}", styles['Normal']))
    
    except Exception as e:
        print(f"⚠️ AI enhancement failed: {e}")
        # Continue without AI enhancement
```

### Step 4: Integrate into Dashboard

Add to `src/dashboard/dashboard.py`:

```python
# At the top of the file
from ai.dashboard_insights import generate_dashboard_insights

# In Dashboard.__init__, add AI insights widget:
def __init__(self, ...):
    # ... existing code ...
    
    # Add AI Insights Panel
    self.setup_ai_insights_panel()

def setup_ai_insights_panel(self):
    """Add AI-powered insights panel to dashboard"""
    ai_card = QFrame()
    ai_card.setStyleSheet("background: white; border-radius: 8px; padding: 15px;")
    ai_layout = QVBoxLayout(ai_card)
    
    # Health Score Widget
    self.health_score_label = QLabel("Health Score: --")
    self.health_score_label.setStyleSheet("font-size: 24px; font-weight: bold; color: #ff6600;")
    ai_layout.addWidget(self.health_score_label)
    
    # Insights List
    self.insights_list = QTextEdit()
    self.insights_list.setReadOnly(True)
    self.insights_list.setMaximumHeight(200)
    ai_layout.addWidget(self.insights_list)
    
    # Add to dashboard layout (adjust position as needed)
    # self.main_layout.addWidget(ai_card)

def update_ai_insights(self):
    """Update AI insights based on current metrics"""
    try:
        # Get current metrics
        current_metrics = {
            'HR': int(self.metric_labels['heart_rate'].text().split()[0]) if 'heart_rate' in self.metric_labels else 70,
            'PR': int(self.metric_labels['pr_interval'].text().split()[0]) if 'pr_interval' in self.metric_labels else 160,
            'QRS': int(self.metric_labels['qrs_duration'].text().split()[0]) if 'qrs_duration' in self.metric_labels else 90,
            'QTc': int(self.metric_labels['qtc_interval'].text().split('/')[1]) if 'qtc_interval' in self.metric_labels else 430,
            'arrhythmias': self.get_detected_arrhythmias()  # Implement this method
        }
        
        # Generate insights
        insights = generate_dashboard_insights(
            current_metrics=current_metrics,
            historical_data=self.get_historical_data(),  # Implement this method
            patient_data=self.user_details
        )
        
        # Update UI
        health_score = insights['health_score']
        self.health_score_label.setText(
            f"Health Score: {health_score['score']}/100 ({health_score['level']})"
        )
        self.health_score_label.setStyleSheet(
            f"font-size: 24px; font-weight: bold; color: {health_score['color']};"
        )
        
        # Update insights text
        insights_html = "<b>Recent Insights:</b><br>"
        for insight in insights['trend_insights'][:3]:  # Show top 3
            insights_html += f"{insight['icon']} {insight['message']}<br>"
        
        if insights['smart_alerts']:
            insights_html += "<br><b>Alerts:</b><br>"
            for alert in insights['smart_alerts'][:2]:  # Show top 2
                insights_html += f"⚠️ {alert['message']}<br>"
        
        self.insights_list.setHtml(insights_html)
    
    except Exception as e:
        print(f"⚠️ Error updating AI insights: {e}")

# Call update_ai_insights periodically (e.g., every 10 seconds)
# In your update loop:
# QTimer.singleShot(10000, self.update_ai_insights)
```

## 📝 Example Usage

### Basic Report Enhancement

```python
from ai.report_enhancer import enhance_report_with_ai

metrics = {
    'HR': 88,
    'PR': 200,
    'QRS': 90,
    'QTc': 430,
    'ST': 0
}

arrhythmias = ['Normal Sinus Rhythm']

enhanced = enhance_report_with_ai(metrics, arrhythmias)

print(enhanced['executive_summary'])
# Output: Professional AI-generated summary

print(enhanced['risk_assessment'])
# Output: {'risk_score': 15, 'risk_level': 'Low', ...}
```

### Dashboard Insights

```python
from ai.dashboard_insights import generate_dashboard_insights

current_metrics = {
    'HR': 88,
    'PR': 200,
    'QRS': 90,
    'QTc': 430,
    'arrhythmias': []
}

insights = generate_dashboard_insights(current_metrics)

print(insights['health_score'])
# Output: {'score': 85, 'level': 'Excellent', ...}

print(insights['smart_alerts'])
# Output: List of context-aware alerts
```

## 🎯 What You Get

### Reports:
- ✅ Professional executive summaries
- ✅ Intelligent, context-aware findings
- ✅ Risk scoring and stratification
- ✅ Plain language explanations
- ✅ Personalized recommendations

### Dashboard:
- ✅ Real-time health score (0-100)
- ✅ Trend analysis and insights
- ✅ Smart, context-aware alerts
- ✅ Personalized recommendations
- ✅ Predictive insights

## 💰 Cost Estimation

### OpenAI GPT-4:
- Report generation: ~$0.03 per report
- Dashboard insights: ~$0.01 per update
- **Monthly for 100 users**: ~$20-50

### Local Models (No API):
- One-time setup: Free (uses existing libraries)
- Ongoing: Free
- **Monthly**: $0

## 🔒 Privacy Options

1. **API-based (OpenAI/Claude)**: 
   - Fastest to implement
   - Requires internet
   - Data sent to API (check privacy policy)

2. **Local models**:
   - Complete privacy
   - Works offline
   - Requires model training

3. **Hybrid**:
   - Use local for sensitive data
   - Use API for complex NLP

## 🚀 Next Steps

1. **Start Simple**: Use API-based solution for quick wins
2. **Test Thoroughly**: Verify AI outputs are accurate
3. **Iterate**: Refine prompts and parameters
4. **Scale**: Move to local models if needed for privacy/cost

## 📚 Resources

- OpenAI API: https://platform.openai.com/docs
- Anthropic Claude: https://docs.anthropic.com/
- Example prompts: See `AI_INTEGRATION_PLAN.md`

---

**Ready to start?** Follow the 4 steps above and you'll have AI-enhanced reports and dashboard in under 10 minutes!


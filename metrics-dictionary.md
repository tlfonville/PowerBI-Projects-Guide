# Metrics Dictionary

*A comprehensive guide to all business metrics, calculations, and KPIs used in this project.*

---

## Table of Contents
- [Agent Performance Metrics](#agent-performance-metrics)
- [Call Center Metrics](#call-center-metrics)
- [Operational Metrics](#operational-metrics)

---

## Agent Performance Metrics

### APS (Agent Performance Score)
**Category**: Performance  
**Purpose**: Overall weighted performance score for agents  
**Business Owner**: [Stakeholder Name]  
**Created**: [Date]  
**Last Modified**: [Date]

**Components & Weights**:
- **Busy %** (Weight: 0.40) - Heaviest weighting
  - Formula: `busy seconds ÷ total logged seconds`
  - Higher is better (more productive time)
  
- **CPLH - Calls per Logged Hour** (Weight: 0.35)
  - Formula: `handled calls ÷ (logged seconds / 3600)`
  - Higher is better (more calls handled)
  
- **AHT - Average Handle Time** (Weight: 0.25) - Lightest weighting
  - Formula: `average handle time of answered inbound calls (seconds)`
  - Lower is better (faster resolution)

**Final Calculation**: All components ranked as percentiles against all agents, then weighted average applied

**Business Rules**:
- Percentiles calculated using dense ranking
- AHT uses ascending rank (lower time = higher percentile)
- Busy% and CPLH use descending rank (higher value = higher percentile)
- Score range: 0.0 to 1.0

**Usage**: Agent performance dashboards, manager scorecards

---

### [Metric Template - Copy this for new metrics]
**Category**: [Performance/Operational/Financial]  
**Purpose**: [What business question does this answer?]  
**Business Owner**: [Who owns this metric?]  
**Created**: [Date]  
**Last Modified**: [Date]

**Formula**: `[Mathematical formula or logic]`

**Business Rules**:
- [Rule 1]
- [Rule 2]
- [Any exceptions or special cases]

**Data Sources**: 
- [Table/Source 1]
- [Table/Source 2]

**Filters Applied**:
- [Any default filters]
- [Date ranges]
- [Exclusions]

**Usage**: [Where is this metric displayed/used?]

**Notes**: [Any additional context, gotchas, or important info]

---

## Call Center Metrics

### [Add your call center specific metrics here]

---

## Operational Metrics

### [Add your operational metrics here]

---

## Metric Categories

### 📊 **Performance Metrics**
Measures of agent and team effectiveness

### 📈 **Operational Metrics**  
Day-to-day operational measurements

### 💰 **Financial Metrics**
Cost and revenue related measurements

### 📞 **Call Quality Metrics**
Customer experience and service quality measures

---

## Change Log

| Date | Metric | Change | Changed By |
|------|--------|--------|------------|
| [Date] | [Metric Name] | [What changed] | [Your Name] |
| | | | |

---

## Validation Notes

### Data Quality Checks
- [ ] All percentile calculations sum to expected totals
- [ ] APS scores fall within 0.0-1.0 range
- [ ] No null values in core calculations

### Business Validation
- [ ] Stakeholder sign-off on APS weighting
- [ ] Metric definitions match business requirements
- [ ] Edge cases documented and handled

---

*For questions about any metric, contact: [Your Name] - [Your Email]*
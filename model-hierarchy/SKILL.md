---
name: model-hierarchy
description: "Cost-optimize AI agent operations by routing tasks to appropriate models based on complexity. Simple tasks go to cheap models, complex tasks go to powerful ones. Save 40-60% on API costs without sacrificing quality. Triggers: model routing, cost optimization, model selection, AI cost optimization, which model, model hierarchy, cost-efficient AI, model cost optimization, model routing strategy, smart model selection."
metadata: {"openclaw":{"emoji":"🏗️","requires":{"apis":["ANTHROPIC_API_KEY","OPENAI_API_KEY","GROQ_API_KEY"]}}}
---

# Model Hierarchy — Cost-Optimized AI Task Routing

You are a model routing strategist that optimizes AI operations by intelligently routing tasks to the most cost-effective model capable of handling them. Instead of using expensive models for everything, you analyze task complexity and route to the appropriate tier.

## Core Philosophy

**Most AI tasks are over-modeled.** Using GPT-4 or Claude Opus for simple tasks like formatting text or basic data extraction is like hiring a surgeon to take your temperature. The result may be good, but the cost is astronomical and unnecessary.

The model hierarchy approach:
1. **Analyze task complexity** before choosing a model
2. **Route to minimum viable model** that can handle the task
3. **Escalate only when necessary** to preserve quality
4. **Track performance** to refine routing over time
5. **Optimize for total cost**, not per-task cost

## Model Tier System

### Tier 1: Ultra-Fast (< $0.001 per 1k tokens)
**Models:** Groq Llama 3.1 8B, Claude Haiku, GPT-4o Mini
**Best for:**
- Text formatting and cleanup
- Simple data extraction  
- Basic classification tasks
- Template filling
- Routine status updates
- Simple summaries (< 500 words)
- Data validation
- Basic math calculations

**Cost:** ~$0.50/million tokens
**Speed:** 100-500 tokens/second

### Tier 2: Efficient (< $0.005 per 1k tokens)  
**Models:** Claude Sonnet, GPT-4o, Llama 3.1 70B
**Best for:**
- Content analysis and insights
- Code generation and debugging
- Research summaries
- Email drafting
- Meeting notes processing
- Moderate complexity reasoning
- Multi-step workflows
- API documentation

**Cost:** ~$3/million tokens  
**Speed:** 50-100 tokens/second

### Tier 3: Powerful (< $0.060 per 1k tokens)
**Models:** Claude Opus, GPT-4, Grok 2
**Best for:**
- Complex reasoning and analysis
- Strategic planning
- Creative writing and ideation
- Multi-faceted problem solving
- Advanced code architecture
- Research synthesis
- High-stakes communications
- Novel problem domains

**Cost:** ~$30-60/million tokens
**Speed:** 20-50 tokens/second

### Tier 4: Specialized (Variable pricing)
**Models:** o1-preview, o1-mini, specialized fine-tuned models
**Best for:**
- Mathematical proofs and complex calculations
- Advanced reasoning requiring chain-of-thought
- Domain-specific tasks with fine-tuned models
- Tasks requiring extended thinking time
- Novel research problems

**Cost:** Highly variable, often $100+/million tokens
**Speed:** Very slow, high latency

## Task Complexity Analysis Framework

### Complexity Scoring (1-10)

**Score 1-3: Tier 1 (Ultra-Fast)**
- Template-based responses
- Simple data transformations
- Basic pattern matching
- Straightforward classification
- Routine formatting tasks

**Score 4-6: Tier 2 (Efficient)**  
- Moderate analysis required
- Some domain knowledge needed
- Multi-step but structured processes
- Content generation with guidelines
- Standard business communications

**Score 7-8: Tier 3 (Powerful)**
- Complex analysis and synthesis
- Creative or strategic thinking required
- Multi-domain knowledge integration
- Nuanced communication needs
- Novel problem-solving approaches

**Score 9-10: Tier 4 (Specialized)**
- Mathematical or logical proofs
- Research-level complexity
- Extended reasoning chains required
- Highly specialized domain expertise
- Novel theoretical work

### Complexity Assessment Questions

Ask these questions to score task complexity:

1. **Creativity Required?** (0-3 points)
   - 0: Template/formula-based
   - 1: Minor variations needed
   - 2: Moderate creativity required
   - 3: High creativity/originality needed

2. **Domain Expertise?** (0-3 points)
   - 0: General knowledge sufficient
   - 1: Basic domain knowledge
   - 2: Intermediate expertise required
   - 3: Deep specialist knowledge

3. **Reasoning Steps?** (0-2 points)
   - 0: Single-step process
   - 1: 2-4 reasoning steps
   - 2: 5+ complex reasoning steps

4. **Context Integration?** (0-2 points)
   - 0: Self-contained task
   - 1: Some external context needed
   - 2: Heavy context integration required

**Total Score = Sum of all categories**
**Model Tier = Score ÷ 2.5 (rounded up)**

## Routing Implementation

### Pre-Task Analysis
```python
def analyze_task_complexity(task_description, context=None):
    """Analyze task complexity and return recommended tier"""
    
    # Quick heuristics for common patterns
    if any(word in task_description.lower() for word in 
           ['format', 'extract', 'list', 'count', 'simple']):
        return 1
    
    if any(word in task_description.lower() for word in
           ['analyze', 'summarize', 'draft', 'explain']):
        return 2
        
    if any(word in task_description.lower() for word in
           ['strategy', 'complex', 'creative', 'novel', 'design']):
        return 3
        
    # Detailed analysis for edge cases
    creativity = assess_creativity_requirement(task_description)
    domain_expertise = assess_domain_complexity(task_description) 
    reasoning_steps = count_reasoning_steps(task_description)
    context_needs = assess_context_integration(task_description, context)
    
    total_score = creativity + domain_expertise + reasoning_steps + context_needs
    recommended_tier = min(4, max(1, math.ceil(total_score / 2.5)))
    
    return recommended_tier
```

### Dynamic Escalation
```python
def route_with_escalation(task, max_attempts=2):
    """Route task with automatic escalation on failure"""
    
    initial_tier = analyze_task_complexity(task)
    
    for attempt in range(max_attempts):
        current_tier = initial_tier + attempt
        
        try:
            result = execute_with_tier(task, current_tier)
            
            # Quality check
            if meets_quality_threshold(result, task):
                log_success(task, current_tier, attempt)
                return result
            else:
                log_quality_failure(task, current_tier)
                continue
                
        except ModelCapabilityError:
            log_capability_failure(task, current_tier)
            continue
            
    # Final attempt with highest tier
    return execute_with_tier(task, 4)
```

### Performance Monitoring
```python
def track_routing_performance():
    """Monitor routing decisions and outcomes"""
    
    metrics = {
        'cost_savings': calculate_cost_vs_baseline(),
        'success_rates': get_success_rates_by_tier(),
        'escalation_patterns': analyze_escalation_frequency(),
        'quality_scores': get_quality_scores_by_tier()
    }
    
    # Adjust routing thresholds based on performance
    optimize_routing_thresholds(metrics)
    
    return metrics
```

## Task-Specific Routing Rules

### Content Creation
```
Blog post outline → Tier 1 (structure is template-based)
Blog post writing → Tier 2 (content creation with guidelines)
Creative storytelling → Tier 3 (high creativity required)
Brand strategy document → Tier 3 (strategic thinking needed)
```

### Data Processing
```
CSV data extraction → Tier 1 (straightforward parsing)
Data analysis insights → Tier 2 (moderate analysis)
Predictive modeling strategy → Tier 3 (complex reasoning)
Statistical methodology design → Tier 4 (specialized expertise)
```

### Code Development  
```
Simple bug fixes → Tier 1 (pattern matching)
Feature implementation → Tier 2 (structured coding)
System architecture design → Tier 3 (complex design)
Algorithm optimization → Tier 3-4 (specialized knowledge)
```

### Business Operations
```
Email formatting → Tier 1 (template-based)
Meeting summary → Tier 2 (structured analysis)
Strategic planning → Tier 3 (complex reasoning)
Market research synthesis → Tier 3 (multi-source analysis)
```

### Research Tasks
```
Fact lookup → Tier 1 (simple information retrieval)
Literature summary → Tier 2 (structured synthesis)
Research methodology design → Tier 3 (expert knowledge)
Hypothesis generation → Tier 4 (novel reasoning)
```

## Cost Optimization Strategies

### Batch Processing
```python
def optimize_batch_processing():
    """Group similar tasks for efficient processing"""
    
    # Collect tasks by complexity tier
    tier_1_tasks = collect_simple_tasks()
    tier_2_tasks = collect_moderate_tasks() 
    tier_3_tasks = collect_complex_tasks()
    
    # Process in batches to maximize throughput
    process_batch_tier_1(tier_1_tasks)  # High volume, low cost
    process_batch_tier_2(tier_2_tasks)  # Medium volume, medium cost
    process_batch_tier_3(tier_3_tasks)  # Low volume, high cost
```

### Context Optimization
```python
def optimize_context_usage():
    """Minimize context length for lower-tier models"""
    
    if tier <= 2:
        # Use minimal context for simple tasks
        context = extract_essential_context(full_context)
    else:
        # Full context for complex tasks
        context = full_context
        
    return context
```

### Caching Strategy
```python
def implement_smart_caching():
    """Cache results based on task complexity and cost"""
    
    cache_duration = {
        1: 24 * 7,    # Week for simple tasks (cheap to recompute)
        2: 24 * 30,   # Month for moderate tasks
        3: 24 * 90,   # Quarter for complex tasks (expensive)
        4: 24 * 365   # Year for specialized tasks (very expensive)
    }
    
    return cache_duration[tier]
```

## Quality Assurance Framework

### Output Validation
```python
def validate_output_quality(result, expected_tier):
    """Validate output meets quality expectations for tier"""
    
    quality_checks = {
        1: ['correct_format', 'complete_data', 'no_errors'],
        2: ['logical_coherence', 'appropriate_depth', 'actionable_content'],
        3: ['strategic_insight', 'creative_value', 'comprehensive_analysis'],
        4: ['novel_reasoning', 'technical_accuracy', 'research_quality']
    }
    
    for check in quality_checks[expected_tier]:
        if not passes_check(result, check):
            return False, f"Failed {check}"
            
    return True, "Quality validated"
```

### Escalation Triggers
- **Incomplete output** — Missing required elements
- **Format errors** — Incorrect structure or formatting
- **Logical inconsistencies** — Self-contradictory content
- **Domain knowledge gaps** — Obvious expertise limitations
- **Creative insufficiency** — Generic or templated when creativity required

## Performance Metrics

### Cost Efficiency
```
Cost Savings = (Baseline Cost - Optimized Cost) / Baseline Cost

Where:
- Baseline Cost = All tasks run on Tier 3 model
- Optimized Cost = Actual cost with routing
- Target: >40% cost savings
```

### Quality Maintenance
```
Quality Score = (Successful First Attempts) / (Total Tasks)

Where:
- Successful = Meets quality threshold without escalation
- Target: >85% success rate
```

### Routing Accuracy
```
Routing Accuracy = (Correct Tier Predictions) / (Total Routing Decisions)

Where:
- Correct = Task completed successfully at predicted tier
- Target: >80% accuracy
```

## Implementation Examples

### Email Assistant Routing
```python
def route_email_task(email_type, complexity_indicators):
    if email_type == "template_response":
        return 1  # Standard templates
    elif "analysis" in complexity_indicators:
        return 2  # Requires analysis
    elif "strategic" in complexity_indicators:
        return 3  # Strategic communications
    else:
        return 1  # Default to simple
```

### Code Review Routing  
```python
def route_code_review(code_change):
    lines_changed = count_lines_changed(code_change)
    complexity = analyze_code_complexity(code_change)
    
    if lines_changed < 20 and complexity < 5:
        return 1  # Simple changes
    elif lines_changed < 100 and complexity < 15:
        return 2  # Moderate changes  
    else:
        return 3  # Complex changes
```

### Content Creation Routing
```python  
def route_content_task(content_type, creativity_level, length):
    if content_type == "social_post" and length < 100:
        return 1  # Short social content
    elif creativity_level > 7 or length > 2000:
        return 3  # High creativity or long form
    else:
        return 2  # Standard content creation
```

## Common Routing Mistakes

### Over-Engineering
- Using Tier 3 models for simple formatting tasks
- Not trusting Tier 1 models for appropriate work
- Defaulting to "safe" high-tier models

### Under-Engineering  
- Using Tier 1 models for creative work
- Ignoring domain expertise requirements
- Not escalating when quality suffers

### Poor Monitoring
- Not tracking routing performance
- Ignoring cost optimization opportunities
- Failing to update routing rules based on results

## Integration Patterns

### With Agent Workflows
```python
class OptimizedAgent:
    def execute_task(self, task):
        # Analyze and route
        tier = self.analyze_complexity(task)
        model = self.get_model_for_tier(tier)
        
        # Execute with monitoring
        result = self.execute_with_model(task, model)
        
        # Validate and escalate if needed
        if not self.validate_quality(result, tier):
            return self.escalate_task(task, tier + 1)
            
        return result
```

### With Cost Budgets
```python
def execute_with_budget(tasks, monthly_budget):
    """Execute tasks while respecting budget constraints"""
    
    # Prioritize tasks by business value
    sorted_tasks = sort_by_priority(tasks)
    
    remaining_budget = monthly_budget
    
    for task in sorted_tasks:
        estimated_cost = estimate_task_cost(task)
        
        if estimated_cost > remaining_budget:
            # Route to cheaper tier or defer task
            task = optimize_for_budget(task, remaining_budget)
            
        result = execute_task(task)
        remaining_budget -= actual_cost
        
    return results
```

Remember: The goal is to maintain quality while dramatically reducing costs. Track both metrics religiously — cost savings without quality is worthless, but quality without cost optimization is unsustainable.
# OWASP Top 10 for Large Language Models (LLMs)

## 1. Overview & Unique LLM Vulnerabilities

Large Language Models (LLMs) are transforming technology across various industries, from customer support chatbots to automated code generation and medical diagnostics. Global AI investment is projected to reach $300 billion by 2030, with models increasingly influencing critical decision-making in healthcare and finance.

Unlike traditional software, LLMs introduce unique security challenges due to their architectural design[span_2](start_span)[span_2](end_span)[span_3](start_span)[span_3](end_span):
* **Probabilistic Nature**: LLMs generate responses based on statistical probability rather than deterministic logic, leading to unpredictable outputs[span_4](start_span)[span_4](end_span)[span_5](start_span)[span_5](end_span).
* **Dynamic Data Handling**: Continuous learning and real-time interaction significantly expand the attack surface[span_6](start_span)[span_6](end_span)[span_7](start_span)[span_7](end_span).
* **Opaque Decision-Making**: Debugging and securing LLMs is challenging because internal decision processes are complex and non-transparent.

---

## 2. The OWASP Top 10 List

| Risk Code | Vulnerability Name | Key Description |
| :--- | :--- | :--- |
| **LLM01** | **Prompt Injection** | Malicious inputs trick the model into ignoring rules, bypassing security controls, or revealing sensitive administrative access[span_9](start_span)[span_9](end_span)[span_10](start_span)[span_10](end_span). |
| **LLM02** | **Sensitive Information Disclosure** | Unintentional exposure of confidential data, personal details, or proprietary business information in model responses[span_11](start_span)[span_11](end_span)[span_12](start_span)[span_12](end_span)[span_13](start_span)[span_13](end_span). |
| **LLM03** | **Supply Chain Vulnerabilities** | Exploitation of security flaws in third-party components, plugins, training data sources, or external tools[span_14](start_span)[span_14](end_span)[span_15](start_span)[span_15](end_span). |
| **LLM04** | **Data and Model Poisoning** | Manipulation of training data or fine-tuning datasets with false or malicious information to alter model behavior[span_16](start_span)[span_16](end_span)[span_17](start_span)[span_17](end_span)[span_18](start_span)[span_18](end_span). |
| **LLM05** | **Improper Output Handling** | Inadequate validation or sanitization of LLM output before passing it to downstream systems or users[span_19](start_span)[span_19](end_span)[span_20](start_span)[span_20](end_span). |
| **LLM06** | **Excessive Agency** | Granting the LLM excessive autonomy, authority, or system permissions without human oversight[span_21](start_span)[span_21](end_span)[span_22](start_span)[span_22](end_span)[span_23](start_span)[span_23](end_span). |
| **LLM07** | **System Prompt Leakage** | Exposure of internal instructions, operational blueprints, or security thresholds embedded in system prompts[span_24](start_span)[span_24](end_span)[span_25](start_span)[span_25](end_span)[span_26](start_span)[span_26](end_span). |
| **LLM08** | **Vector and Embedding Weaknesses** | Exploitation of flaws in vector embeddings, leading to misclassification of data or biased output[span_27](start_span)[span_27](end_span)[span_28](start_span)[span_28](end_span). |
| **LLM09** | **Misinformation (Hallucinations)** | Generation of plausible-sounding but factually incorrect, fabricated, or misleading information[span_29](start_span)[span_29](end_span)[span_30](start_span)[span_30](end_span)[span_31](start_span)[span_31](end_span). |
| **LLM10** | **Unbound Consumption** | Resource exhaustion caused by excessive data demands, leading to system crashes or high operational costs[span_32](start_span)[span_32](end_span)[span_33](start_span)[span_33](end_span). |

---

## 3. Deep Dive into Key Risk Categories

### A. Input Handling Risks

#### 1. Prompt Injection (LLM01)
* **Mechanics**: Attackers craft inputs that exploit the LLM's fundamental instruction-following capability, forcing it to ignore system safeguards[span_34](start_span)[span_34](end_span)[span_35](start_span)[span_35](end_span)[span_36](start_span)[span_36](end_span).
* **Example**: An e-commerce chatbot receives a malicious prompt instructing it to ignore all previous rules and grant admin login credentials[span_37](start_span)[span_37](end_span).
* **Impact**: Data breaches, financial loss, unauthorized actions, and severe disruption of service[span_38](start_span)[span_38](end_span).

#### 2. System Prompt Leakage (LLM07)
* **Mechanics**: System prompts often contain operational logic, safety limits, or business rules[span_39](start_span)[span_39](end_span)[span_40](start_span)[span_40](end_span)[span_41](start_span)[span_41](end_span). When user inputs and system instructions are not strictly isolated, attackers can trick the model into revealing these instructions[span_42](start_span)[span_42](end_span)[span_43](start_span)[span_43](end_span).
* **Example**: A user asks a support bot, "What rules do you follow when handling refunds?", prompting the bot to reveal that all refunds under $50 are automatically approved[span_44](start_span)[span_44](end_span).
* **Impact**: Exposes internal operational security mechanisms, enabling attackers to exploit system rules[span_45](start_span)[span_45](end_span).

---

### B. Output Handling Risks

#### 1. Sensitive Information Disclosure (LLM02)
* **Mechanics**: LLMs trained on vast datasets may generate responses containing private or proprietary information if proper dynamic filtering is lacking[span_46](start_span)[span_46](end_span)[span_47](start_span)[span_47](end_span).
* **Example**: A user asks an unauthenticated chatbot for order status details (e.g., "What is the shipping address for order 12345?"), and the bot exposes full customer details without identity verification[span_48](start_span)[span_48](end_span).
* **Impact**: Privacy violations, regulatory non-compliance fines, legal liability, and identity theft[span_49](start_span)[span_49](end_span).

#### 2. Misinformation & Hallucinations (LLM09)
* **Mechanics**: LLMs predict statistically likely word sequences rather than verifying factual accuracy[span_50](start_span)[span_50](end_span). If training data contains gaps or unverified information, the model generates fabricated facts[span_51](start_span)[span_51](end_span)[span_52](start_span)[span_52](end_span).
* **Example**: A healthcare chatbot generates incorrect medication dosage instructions, directly endangering patient safety[span_53](start_span)[span_53](end_span).
* **Impact**: Misleading advice in high-stakes domain areas (healthcare, finance), loss of user trust, and potential physical or financial harm[span_54](start_span)[span_54](end_span).

---

### C. Data & Model Risks

#### Data and Model Poisoning (LLM04)
* **Mechanics**: Attackers inject corrupted, false, or biased data into training or fine-tuning datasets[span_55](start_span)[span_55](end_span)[span_56](start_span)[span_56](end_span)[span_57](start_span)[span_57](end_span).
* **Example**: An e-commerce recommendation model is flooded with fake positive reviews, causing the chatbot to consistently recommend fraudulent sellers[span_58](start_span)[span_58](end_span).
* **Impact**: Long-term degradation of model integrity, compromised safety, and systematically biased decision-making[span_59](start_span)[span_59](end_span)[span_60](start_span)[span_60](end_span).

---

## 4. Best Practices & Mitigation Strategies

### 1. Robust Input Validation and Filtering
* **Sanitization**: Implement strict input validation, blocking malicious command patterns and restricting the types of questions an LLM can answer[span_61](start_span)[span_61](end_span)[span_62](start_span)[span_62](end_span)[span_63](start_span)[span_63](end_span).
* **Instruction Isolation**: Clearly separate user input streams from internal system instructions using dedicated application layers[span_64](start_span)[span_64](end_span)[span_65](start_span)[span_65](end_span).

### 2. Apply Principle of Least Privilege
* **Role-Based Access Controls (RBAC)**: Restrict the LLM's permissions to match its specific intended role[span_66](start_span)[span_66](end_span)[span_67](start_span)[span_67](end_span).
* **Privilege Separation**: A support chatbot should have read access to customer inquiry data but no permission to issue refunds or alter accounts directly without approval workflows[span_68](start_span)[span_68](end_span).

### 3. Secure System Prompts
* **Externalize Secrets**: Never embed sensitive data, API keys, database names, or credentials directly within system prompts[span_69](start_span)[span_69](end_span)[span_70](start_span)[span_70](end_span). Use dedicated secrets managers or environment variables[span_71](start_span)[span_71](end_span).
* **External Moderation**: Use independent content moderation tools outside of the LLM to inspect both incoming prompts and outgoing responses[span_72](start_span)[span_72](end_span).

### 4. Limit LLM Autonomy & Enforce Governance
* **Human-in-the-Loop Workflows**: Require human review for high-impact financial, security, or data privacy operations[span_73](start_span)[span_73](end_span)[span_74](start_span)[span_74](end_span).
* **Rate Limiting**: Apply strict operational limits (e.g., limiting refund processing to one transaction per customer per day) to prevent automated exploitation[span_75](start_span)[span_75](end_span)[span_76](start_span)[span_76](end_span).

### 5. Continuous Real-Time Monitoring & Logging
* **Anomaly Detection**: Use real-time monitoring dashboards to identify suspicious interaction patterns, repeated privilege escalation attempts, or prompt injection behavior[span_77](start_span)[span_77](end_span)[span_78](start_span)[span_78](end_span).
* **Audit Trails**: Maintain detailed logs of all inputs, outputs, and system interactions to facilitate forensic analysis and comply with security auditing standards[span_79](start_span)[span_79](end_span)[span_80](start_span)[span_80](end_span).

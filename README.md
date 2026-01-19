# ember-guard
A production-grade hierarchical machine learning framework for intelligent static malware detection and classification of Windows Portable Executable files, leveraging ensemble learning, confidence-aware decision-making, and progressive taxonomic classification to achieve state-of-the-art accuracy while maintaining operational efficiency at scale.
Overview
Modern malware has evolved into a sophisticated threat landscape characterized by polymorphic behavior, advanced obfuscation techniques, and evasion mechanisms specifically designed to bypass traditional signature-based antivirus solutions. This project addresses these challenges through a comprehensive multi-stage ensemble learning system that combines the strengths of multiple gradient boosting algorithms with hierarchical classification strategies and confidence-aware calibration mechanisms.
The framework processes Windows Portable Executable (PE) files through a progressive decision pipeline, beginning with binary malware detection, advancing through semantic umbrella categorization, and culminating in fine-grained malware family identification. This hierarchical approach not only improves classification accuracy but also reduces error propagation, enhances interpretability, and enables efficient computational resource allocation by terminating benign samples early in the pipeline.
Trained and validated on over 3.1 million samples from the combined EMBER 2018 and EMBER 2024 datasets, the system demonstrates exceptional performance under realistic class imbalance conditions where benign files substantially outnumber malicious specimens—a critical consideration for real-world deployment scenarios. The integration of temporal diversity across multiple dataset vintages ensures robustness against concept drift, addressing the persistent challenge of model degradation as malware characteristics evolve over time.
Core Architecture
Multi-Stage Detection Pipeline
The system architecture implements a four-stage hierarchical classification pipeline, each stage progressively narrowing the decision space and refining classification granularity:
Stage 1: Binary Malware Detection
The foundation of the pipeline employs a heterogeneous ensemble comprising ExtraTrees, XGBoost, and LightGBM classifiers. Each model independently analyzes 534 carefully selected static features extracted from the input PE file and produces a probability estimate representing the likelihood of malicious behavior. These predictions are dynamically aggregated using confidence-weighted voting, where classifiers demonstrating higher certainty contribute more substantially to the final decision. Post-hoc probability calibration is applied to suppress mid-confidence false positives while preserving detection sensitivity for high-risk samples. Samples classified as benign are immediately terminated, significantly reducing computational overhead for the majority of clean files encountered in operational environments.
Stage 2: Umbrella Classification
Malicious samples advance to umbrella categorization, where they are grouped into high-level semantic categories including Trojan, Stealer, Ransomware, Miner, Backdoor, Virus, and Worm. This stage leverages both machine learning classification and rule-based taxonomic mapping derived from established antivirus naming conventions, ensuring alignment with industry-standard malware taxonomies. By constraining the label space at this early stage, the system reduces downstream classification complexity and limits error propagation across subsequent stages.
Stage 3: Trojan Sub-Umbrella Classification
Given the prevalence and diversity of Trojan-based malware, samples assigned to the Trojan umbrella undergo additional sub-categorization into specialized classes including Loader, Banker, Spyware, Remote Access Trojan (RAT), and Generic Trojan variants. This intermediate stage captures structural and behavioral distinctions within the broad Trojan category, enabling more precise threat characterization and improved downstream family classification accuracy.
Stage 4: Malware Family Classification
The final stage performs fine-grained classification across 75 distinct malware families using a LightGBM-based multi-class classifier. Only families with sufficient sample representation are included to ensure stable learning and reliable predictions. The hierarchical filtering applied in previous stages substantially reduces label confusion among structurally similar families, resulting in high precision even for minority classes.
Technical Implementation
Feature Engineering and Extraction
The system employs the standardized EMBER feature schema, extracting a comprehensive 534-dimensional feature vector from each PE file. These features encompass:

PE Header Metadata: Structural information including machine type, timestamp, subsystem, and compilation characteristics
Section Statistics: Size, entropy, virtual addresses, and characteristics of executable sections
Import Analysis: Functions imported from system libraries, revealing API usage patterns
Byte Histograms: Statistical distribution of byte values across the executable
Entropy Measurements: Information-theoretic metrics indicating compression or obfuscation

This rich feature representation captures both structural properties and behavioral indicators observable through static analysis alone, enabling effective detection without requiring runtime execution or sandbox environments.
Ensemble Learning Strategy
The heterogeneous ensemble design leverages complementary strengths of multiple classifier architectures:
ExtraTrees (Extremely Randomized Trees) contributes variance reduction through aggressive randomization of both feature selection and split points, improving generalization and reducing overfitting on noisy features.
XGBoost (Extreme Gradient Boosting) excels at capturing complex non-linear feature interactions through regularized gradient boosting, providing strong discriminative power for subtle malicious patterns.
LightGBM (Light Gradient Boosting Machine) delivers computational efficiency through histogram-based learning and leaf-wise tree growth, enabling scalable training on the massive 3.1M+ sample dataset while maintaining competitive accuracy.
Rather than simple majority voting, the ensemble employs dynamic confidence-weighted aggregation where each classifier's contribution is proportional to its prediction certainty for the specific sample under evaluation. This adaptive weighting mechanism proves particularly effective near decision boundaries where individual classifiers may exhibit varying levels of uncertainty.
Confidence-Aware Calibration
Raw probability outputs from machine learning classifiers are often poorly calibrated, particularly for gradient boosting models trained on imbalanced datasets. The system implements post-hoc probability calibration using isotonic regression to map raw ensemble scores to well-calibrated probability estimates. This calibration process enables the assignment of samples to interpretable risk bands:

Likely Benign: High-confidence benign classification, analysis terminated
Low-Confidence Suspicious: Borderline samples requiring additional scrutiny
Suspicious Win32: Moderate-confidence malware detections
High-Risk Malware: High-confidence malicious classifications

This stratification supports operational workflows where security analysts can prioritize investigation efforts based on calibrated risk assessments rather than arbitrary threshold-based classifications.
Go-Based Heuristic Validation
To enhance robustness beyond pure machine learning, the system integrates a lightweight static heuristic engine implemented in Go. This component evaluates rule-based indicators including abnormal section counts, suspicious entropy patterns, known malicious import combinations, and invalid header configurations. The heuristic layer operates in parallel with machine learning classification, providing an additional validation mechanism that can flag samples exhibiting clear structural anomalies even when machine learning confidence is moderate.
Performance and Scalability
Benchmark Results
Extensive evaluation on the combined EMBER 2018 and EMBER 2024 dataset demonstrates exceptional performance:

Binary Detection ROC-AUC: 96%+ across all experimental configurations
Overall Accuracy: 96.71% for complete pipeline including family classification
Macro F1-Score: 0.9662, indicating balanced per-family performance despite severe class imbalance
Weighted F1-Score: 0.9670, reflecting strong aggregate system performance
False Positive Reduction: Significant suppression of mid-confidence false alarms through calibration

The heterogeneous ensemble consistently outperforms individual classifiers across all metrics, validating the effectiveness of classifier diversity and confidence-weighted aggregation strategies.
Computational Efficiency
The hierarchical architecture delivers substantial computational benefits in operational deployment:

Early Termination: Benign samples (typically 70-80% of real-world traffic) are classified and terminated at Stage 1, avoiding unnecessary downstream processing
Progressive Complexity: Computational resources scale with threat severity, with only confirmed malware advancing through expensive family classification
Inference Speed: Optimized feature extraction and model inference support high-throughput scanning suitable for enterprise gateway deployment
Training Scalability: Linear scaling of training time with dataset size enables incorporation of emerging malware samples through incremental retraining

Temporal Robustness
By training on the combined EMBER 2018 and EMBER 2024 datasets, the system demonstrates improved resilience against concept drift—the phenomenon where malware characteristics evolve over time, degrading model performance. Exposure to temporally diverse samples spanning multiple years ensures the learned feature representations generalize across evolving threat landscapes, a critical property for maintaining detection effectiveness in long-term deployment.
Use Cases and Applications
This framework addresses multiple deployment scenarios across the cybersecurity operational spectrum:
Enterprise Gateway Protection: Real-time scanning of executables at network perimeters, email gateways, and download checkpoints with calibrated risk scoring for automated policy enforcement.
Cloud Security Services: Scalable malware analysis infrastructure for cloud-based security platforms requiring high-throughput classification with minimal latency.
Threat Intelligence Research: Fine-grained family classification supports malware research, campaign attribution, and threat actor profiling for advanced persistent threat (APT) investigations.
Security Operations Centers (SOCs): Confidence-aware verdicts enable intelligent alert prioritization, reducing analyst workload by focusing attention on high-confidence detections and ambiguous borderline cases.
Incident Response: Rapid static triage of suspicious executables during active incident investigations without requiring time-consuming dynamic analysis or sandbox execution.
Future Directions
While the current implementation focuses exclusively on static analysis, several promising extensions can further enhance system capabilities:
Hybrid Analysis Integration: Incorporating dynamic behavioral features alongside static characteristics would improve detection of heavily obfuscated or packed malware observable only at runtime.
Continual Learning Mechanisms: Implementing incremental learning strategies would enable continuous adaptation to emerging malware families without expensive full retraining cycles.
Cross-Platform Support: Extending beyond Windows PE files to support ELF binaries, Mach-O executables, Android APKs, and other file formats would broaden applicability across diverse computing environments.
Adversarial Robustness: Incorporating adversarial training and robust feature selection would improve resilience against adversarial evasion attacks specifically crafted to fool machine learning detectors.
Model Compression: Applying knowledge distillation and pruning techniques would enable deployment on resource-constrained edge devices and embedded security appliances.
Research Foundation
This implementation synthesizes best practices from contemporary malware detection research, including ensemble learning theory, hierarchical classification strategies, probability calibration techniques, and class imbalance handling methodologies. The system was developed through academic research at Vishwakarma Institute of Technology, Pune, and reflects rigorous experimental validation on industry-standard benchmark datasets.

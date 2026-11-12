# CAN and Automotive Intrusion Detection: Overview

This document introduces the communication and security concepts that
underpin the CANDICE research lab. It covers the Controller Area Network
(CAN), common automotive network threats, intrusion detection systems,
and the main approaches used to detect suspicious CAN traffic.

------------------------------------------------------------------------

## 1. Automotive In-Vehicle Networks

Modern vehicles contain many electronic control units (ECUs). An ECU is
an embedded computer responsible for one or more vehicle functions, such
as engine management, braking, steering, body electronics, or driver
assistance.

ECUs exchange information over in-vehicle networks. These networks allow
distributed components to coordinate without requiring every component
to have a direct connection to every other component.

Common in-vehicle communication technologies include:

-   **CAN (Controller Area Network):** A widely used message-oriented
    bus for communication among automotive ECUs.
-   **LIN (Local Interconnect Network):** A lower-cost network commonly
    used for simpler body and comfort functions.
-   **FlexRay:** A deterministic communication network designed for
    applications with more demanding timing requirements.
-   **Automotive Ethernet:** A high-bandwidth networking technology used
    in newer vehicle architectures and data-intensive applications.

A vehicle may use several network technologies connected through
gateways. CAN remains an important target for automotive cybersecurity
research because of its broad use and its original design priorities of
reliable, efficient real-time communication rather than modern network
security.

## 2. Controller Area Network (CAN)

### 2.1 What is CAN?

CAN is a serial communication protocol originally developed for embedded
control systems and widely adopted in vehicles. It allows multiple ECUs
to communicate over a shared bus.

CAN is **message-oriented**, not address-oriented in the same way as an
IP network. A CAN frame carries an identifier that describes the
message's meaning or priority. ECUs decide which messages to receive and
process based on their configuration and application logic.

In a typical high-speed CAN physical network, ECUs connect to a
differential bus consisting of two signal lines:

-   **CAN_H:** CAN high.
-   **CAN_L:** CAN low.

The differential signaling helps the network tolerate electrical noise.
The bus is commonly terminated at both physical ends with resistors,
typically 120 ohms each, though actual network design depends on the
topology and transceiver requirements.

### 2.2 How CAN communication works

CAN uses a multi-master bus. Any node may attempt to transmit when the
bus is available. If multiple nodes begin transmitting at the same time,
bitwise arbitration determines which message continues.

CAN uses two logical bus states:

  Bus state         Meaning
  ----------------- ----------------------------------
  Dominant (`0`)    Overrides a recessive bit
  Recessive (`1`)   Does not override a dominant bit

During arbitration, a node monitors the bus while transmitting. If it
sends a recessive bit but observes a dominant bit, it stops transmitting
and retries later. This mechanism is non-destructive: the message that
wins arbitration continues without being corrupted by the losing
transmitters.

For identifiers of the same frame format, a numerically lower identifier
generally has higher arbitration priority because it presents dominant
bits earlier. The identifier is therefore used for both message
identification and bus arbitration; it is not inherently a source
address or proof of sender identity.

### 2.3 CAN frame structure

A Classical CAN data frame contains fields that support arbitration,
control, payload delivery, and error detection.

  -----------------------------------------------------------------------
  Field                               Purpose
  ----------------------------------- -----------------------------------
  Start of Frame (SOF)                Marks the beginning of a frame

  Arbitration field                   Contains the identifier and
                                      arbitration-related control bit(s)

  Control field                       Includes control information and
                                      the data length code

  Data field                          Carries the application payload

  CRC field                           Supports detection of transmission
                                      errors

  ACK field                           Allows receivers to acknowledge
                                      correct frame reception at the link
                                      layer

  End of Frame (EOF)                  Marks the end of the frame
  -----------------------------------------------------------------------

Two common identifier formats are used:

  Format                    Identifier length
  ----------------------- -------------------
  Standard (base) frame               11 bits
  Extended frame                      29 bits

Classical CAN carries up to **8 data bytes** in a data frame. CAN FD
extends the protocol to support larger payloads of up to **64 data
bytes** and a faster data phase when the network and nodes support it.
CAN FD is not interchangeable with Classical CAN on a network unless the
participating equipment and configuration support the relevant protocol
mode.

### 2.4 CAN message identifiers and payloads

A CAN message is commonly represented by:

-   **CAN ID:** The identifier associated with the message.
-   **DLC:** The data length code, which indicates the payload length
    within the protocol's rules.
-   **Data:** The payload bytes.
-   **Timestamp:** Often added by a capture device or software tool; it
    is not a standard field transmitted in the CAN frame.
-   **Bus/channel metadata:** Often added by logging tools to identify
    the interface or network.

For example, a logged frame might be represented as:

``` text
Timestamp       CAN ID   DLC   Data
12.345678       0x123    8     10 2A 00 7F 00 00 00 00
```

The meaning of a CAN identifier and its payload is usually defined by
the vehicle's communication design, database, or application software. A
raw payload byte does not have a universal meaning across all vehicles.
Signal definitions may specify scaling, offsets, signedness, bit
positions, units, counters, or checksums.

### 2.5 CAN error handling is not cybersecurity

CAN includes mechanisms such as CRC checks, bit monitoring,
acknowledgement, frame checks, and error counters. These mechanisms help
detect transmission faults and manage faulty nodes.

They do **not** establish that a message came from a trusted ECU. A
syntactically valid frame can still contain malicious or unsafe data.
Link-layer error handling should therefore not be confused with
authentication, authorization, or intrusion detection.

## 3. Automotive CAN Security

### 3.1 Why CAN needs security monitoring

The original CAN protocol was designed for dependable communication
among trusted electronic components. Classical CAN does not provide
built-in cryptographic authentication or encryption for ordinary frames.

Depending on the vehicle architecture and surrounding protections, an
attacker who gains access to a CAN segment may be able to:

-   Observe messages exchanged between ECUs.
-   Inject frames using selected identifiers.
-   Replay previously captured messages.
-   Transmit at a high rate and interfere with normal communication.
-   Attempt to impersonate the expected behavior of another ECU.

Modern vehicles may add security controls at gateways, ECUs, diagnostic
interfaces, or higher protocol layers. The actual exposure depends on
the vehicle, its network segmentation, hardware, software, and security
configuration. A CAN IDS is one possible layer of defense, not a
substitute for secure architecture.

### 3.2 Common CAN-related attack categories

The following categories are commonly discussed in CAN security
research. Their feasibility and impact depend on network access, vehicle
design, and the attacker's capabilities.

  -----------------------------------------------------------------------
  Attack category         General description     Potential observable
                                                  behavior
  ----------------------- ----------------------- -----------------------
  Denial of Service (DoS) Attempts to prevent     Unusually high traffic,
                          legitimate              repeated high-priority
                          communication or        frames, missed expected
                          consume bus capacity    messages

  Fuzzing                 Sends malformed,        Unusual IDs, unexpected
                          random, or unexpected   payload patterns,
                          identifiers and         protocol violations
                          payloads                

  Replay                  Retransmits previously  Repeated or stale
                          observed messages       message sequences,
                                                  timing inconsistencies

  Spoofing /              Sends frames that       Unexpected message
  impersonation           appear to represent     timing, conflicting
                          another message source  values, implausible
                          or function             signal behavior

  Fabrication / injection Inserts frames that     New or unexpected
                          were not expected in    frames, abnormal
                          the current context     frequency, invalid
                                                  context

  Masquerade              Attempts to imitate the Subtle timing,
                          normal traffic pattern  sequence, counter, or
                          of a legitimate ECU     signal inconsistencies
  -----------------------------------------------------------------------

Attack labels and definitions can vary between datasets and
publications. An experiment should state exactly how its dataset defines
each attack class.

## 4. Intrusion Detection Systems (IDS)

### 4.1 What is an IDS?

An **Intrusion Detection System (IDS)** monitors activity in a computer
system or network to identify behavior that may indicate an attack,
misuse, or policy violation.

An IDS observes activity, analyzes it, and produces an alert or
classification when it detects suspicious behavior. Depending on its
design, it may use rules, signatures, statistical thresholds, or
machine-learning models.

An IDS generally does not block traffic by itself. Its primary role is
detection and reporting.

### 4.2 IDS versus IPS

An **Intrusion Prevention System (IPS)** goes beyond detection by taking
action to prevent or contain activity judged to be malicious. Possible
actions include dropping traffic, blocking a source, or triggering
another response.

  -----------------------------------------------------------------------
  Capability              IDS                     IPS
  ----------------------- ----------------------- -----------------------
  Monitor activity        Yes                     Yes

  Detect suspicious       Yes                     Yes
  behavior                                        

  Generate alerts         Yes                     Yes

  Automatically block or  Not normally            Depending on design
  modify traffic                                  

  Main concern            Detection quality and   Detection plus safe,
                          visibility              timely response
  -----------------------------------------------------------------------

In automotive systems, prevention requires particular care. Blocking a
legitimate safety-related message or introducing excessive latency can
itself create a hazard. A research CAN IDS should not be described as an
IPS unless it actually implements and validates a prevention mechanism.

### 4.3 Where can a CAN IDS operate?

A CAN IDS can be placed at different points in a vehicle network:

-   **Bus-level monitoring:** Observes traffic on a CAN segment through
    a suitable interface.
-   **Gateway-based monitoring:** Inspects traffic as it passes between
    network segments.
-   **ECU-level monitoring:** Runs on or alongside an ECU and monitors
    accessible communication.
-   **Off-board analysis:** Processes recorded CAN traces after
    collection rather than monitoring a live vehicle network.

The deployment location affects what traffic is visible, what resources
are available, and how quickly the system can respond.

### 4.4 CAN IDS processing pipeline

A typical data-driven CAN IDS follows a sequence such as:

``` text
CAN traffic / recorded trace
          |
          v
     Data collection
          |
          v
  Parsing and validation
          |
          v
 Preprocessing and features
          |
          v
   Detection algorithm
          |
          v
 Classification / anomaly score
          |
          v
    Alert or log output
          |
          v
 Evaluation and analysis
```

The exact pipeline varies. A rule-based detector may not require model
training, while a supervised learning system requires labeled examples.
An anomaly detector may learn a baseline of normal traffic and flag
deviations.

## 5. CAN Intrusion Detection Approaches

### 5.1 Rule-based and specification-based detection

These methods compare observed traffic against defined rules or expected
communication behavior.

Examples include:

-   Checking whether an identifier is permitted on a segment.
-   Checking whether a message appears within an expected time interval.
-   Checking whether payload values fall within defined bounds.
-   Checking counters, rolling sequences, or known message formats.

**Strengths:** Interpretable, can be computationally efficient, and does
not always require a training dataset.

**Limitations:** Rules require reliable specifications and maintenance.
Unknown attacks or changes in legitimate vehicle behavior may not be
covered.

### 5.2 Signature-based detection

Signature-based systems look for patterns associated with known attacks
or known malicious activity.

**Strengths:** Can be effective for recognized patterns and can produce
understandable alerts.

**Limitations:** Detection depends on available signatures. New or
modified attacks may evade signatures until the rules are updated.

### 5.3 Statistical and timing-based detection

These methods model properties such as message frequency, inter-arrival
time, identifier occurrence, or changes in traffic volume.

For example, a detector may flag a CAN identifier that appears much more
frequently than its learned or specified baseline.

**Strengths:** Can be lightweight and useful for identifying traffic
anomalies.

**Limitations:** Legitimate operating modes can change traffic patterns.
Threshold selection and baseline quality are important.

### 5.4 Machine-learning-based detection

Machine-learning approaches learn a mapping from extracted traffic
features to labels or anomaly scores.

Common supervised algorithms include:

-   Decision Trees and Random Forests.
-   Support Vector Machines (SVM).
-   k-Nearest Neighbors (k-NN).
-   Gradient-boosted trees.
-   Logistic Regression and other linear classifiers.

**Strengths:** Can model relationships among multiple features and
provide strong baselines for structured data.

**Limitations:** Supervised methods depend on representative labeled
data. Dataset bias, class imbalance, leakage, and distribution changes
can lead to misleading performance.

### 5.5 Deep-learning-based detection

Deep-learning methods use neural networks to learn representations from
CAN traffic features or sequences.

Examples include:

  -----------------------------------------------------------------------
  Model family                        Typical use
  ----------------------------------- -----------------------------------
  Multilayer Perceptron (MLP)         Classification using tabular or
                                      engineered features

  Convolutional Neural Network (CNN)  Learning local patterns from
                                      structured inputs or sequences

  Recurrent Neural Network (RNN)      Modeling sequential traffic
                                      behavior

  LSTM / GRU                          Learning temporal dependencies in
                                      message sequences

  Autoencoder                         Learning normal patterns and using
                                      reconstruction error for anomaly
                                      detection
  -----------------------------------------------------------------------

Deep-learning models can capture complex relationships, but they may
require more data, computation, and careful validation than simpler
methods.

### 5.6 TinyML and edge-based detection

TinyML explores machine-learning inference on devices with limited
memory, compute, and power budgets. In a CAN IDS context, the objective
may be to run a compact detector close to the data source.

Common optimization techniques include:

-   **Quantization:** Represents weights and/or activations with lower
    precision, such as INT8.
-   **Quantization-Aware Training (QAT):** Simulates quantization
    effects during training to help the model adapt to lower-precision
    inference.
-   **Post-Training Quantization (PTQ):** Quantizes a trained model
    after training.
-   **Pruning:** Removes selected parameters or structures to reduce
    model complexity.
-   **Knowledge distillation:** Trains a smaller model to reproduce
    behavior from a larger teacher model.

Optimization should be evaluated on both detection performance and
deployment characteristics. A smaller model is not automatically faster
on every hardware target; latency and memory use should be measured on
the intended device.

## 6. Data Representation and Feature Engineering

A CAN IDS does not necessarily operate directly on raw frames. Many
systems transform captured traffic into features suitable for rules or
models.

Potential features include:

  -----------------------------------------------------------------------
  Feature                             Description
  ----------------------------------- -----------------------------------
  CAN identifier                      Identifier associated with the
                                      frame

  DLC                                 Payload length information

  Payload bytes                       Individual data bytes in the frame

  Inter-arrival time                  Time elapsed since a previous frame
                                      or relevant event

  Message frequency                   Number of occurrences over a time
                                      window

  Identifier sequence                 Order or pattern of observed
                                      identifiers

  Payload statistics                  Byte distributions or changes over
                                      time

  Signal values                       Decoded physical or logical values,
                                      when signal definitions are
                                      available

  Sequence/counter fields             Application-level counters or
                                      rolling values, when known
  -----------------------------------------------------------------------

Feature selection should reflect the detection objective and the
information actually available at deployment time. A feature that
depends on a proprietary signal database may not be available in a
generic deployment environment.

### 6.1 Window-based and sequence-based inputs

Traffic can be represented as individual frames or as groups of frames.

-   **Frame-level input:** Each frame is classified independently using
    its identifier, length, and payload.
-   **Window-based input:** A fixed number of frames or a time interval
    is grouped to capture local traffic behavior.
-   **Sequence-based input:** An ordered sequence is provided to a model
    that can learn temporal relationships.

The representation affects the model's ability to detect timing,
ordering, and multi-frame attacks. It also affects inference latency and
memory requirements.

### 6.2 Preprocessing considerations

Common preprocessing operations include:

1.  Parsing raw logs into structured records.
2.  Validating identifiers, payload lengths, and timestamps.
3.  Handling missing, malformed, or duplicated records.
4.  Extracting or encoding features.
5.  Encoding class labels, where supervised learning is used.
6.  Scaling or normalizing features when required by the model.
7.  Splitting data into training, validation, and test sets.

Preprocessing parameters must be learned from the training data only
where applicable. For example, a scaler should be fitted on the training
partition and then applied unchanged to validation and test data. This
helps prevent data leakage.

## 7. Evaluating a CAN IDS

Accuracy alone is not sufficient to evaluate an intrusion detector,
especially when normal traffic greatly outnumbers attack traffic.

### 7.1 Classification metrics

For binary classification, the usual confusion-matrix terms are:

  -----------------------------------------------------------------------
  Term                                Meaning
  ----------------------------------- -----------------------------------
  True Positive (TP)                  An attack is correctly detected

  True Negative (TN)                  Normal traffic is correctly
                                      identified

  False Positive (FP)                 Normal traffic is incorrectly
                                      flagged as an attack

  False Negative (FN)                 An attack is missed
  -----------------------------------------------------------------------

Common metrics include:

  ---------------------------------------------------------------------------------------------------
  Metric                  Formula                                             Interpretation
  ----------------------- --------------------------------------------------- -----------------------
  Accuracy                `(TP + TN) / (TP + TN + FP + FN)`                   Overall fraction of
                                                                              correct predictions

  Precision               `TP / (TP + FP)`                                    Fraction of attack
                                                                              alerts that are correct

  Recall                  `TP / (TP + FN)`                                    Fraction of attacks
                                                                              that are detected

  F1-score                `2 × (Precision × Recall) / (Precision + Recall)`   Harmonic mean of
                                                                              precision and recall

  False Positive Rate     `FP / (FP + TN)`                                    Fraction of normal
                                                                              samples incorrectly
                                                                              flagged
  ---------------------------------------------------------------------------------------------------

For multi-class problems, report per-class precision, recall, and
F1-score, and state how macro, micro, or weighted averages are
calculated. Include a confusion matrix where useful.

### 7.2 Operational and deployment metrics

For embedded or real-time systems, detection metrics should be
complemented by measurements such as:

-   Model size on disk or in flash.
-   RAM and peak memory use.
-   Inference latency and throughput.
-   CPU utilization and power consumption, where measurable.
-   End-to-end detection delay, including collection and preprocessing.
-   Resource use under realistic traffic load.

Measurements should identify the hardware, software, input conditions,
and measurement method. Results from a desktop environment should not be
presented as embedded-device performance.

### 7.3 Evaluation pitfalls

A high test score does not necessarily mean a detector will work on a
real vehicle. Common pitfalls include:

-   **Data leakage:** Information from the test set influences training
    or preprocessing.
-   **Non-independent splits:** Highly similar or adjacent records
    appear in both training and test sets.
-   **Class imbalance:** Overall accuracy hides poor detection of
    minority attack classes.
-   **Dataset-specific patterns:** A model learns artifacts of a capture
    setup or dataset rather than general attack behavior.
-   **Unrealistic timing:** Offline classification results are treated
    as proof of real-time capability.
-   **Unrepresentative conditions:** The test data does not cover
    different vehicle states, traffic loads, or operating modes.

Evaluation should state the split strategy and limitations clearly.
Where possible, test on separate captures, sessions, vehicles, or
operating conditions to assess generalization.

## 8. Scope of the CANDICE Lab

CANDICE is a versioned research collection for experimenting with CAN
intrusion detection methods and their practical constraints.

The lab is organized around independent implementations. Each version
should document its own:

-   Detection objective and approach.
-   Dataset and preprocessing pipeline.
-   Model or rule configuration.
-   Dependencies and execution instructions.
-   Evaluation protocol and measured results.
-   Limitations and deployment assumptions.

The first implementation explores a TinyML-oriented neural-network
workflow with quantization-aware training and INT8 TensorFlow Lite
conversion. Future versions may explore other detection approaches,
architectures, datasets, or deployment targets.

The repository is intended for research and defensive experimentation.
It does not claim that any implementation is suitable for deployment in
a safety-critical vehicle without additional validation, engineering,
and certification.

## 9. Further Reading

The following standards and resources provide useful background. Always
consult the applicable edition and implementation documentation for
technical work.

-   **ISO 11898 series** --- Road vehicles --- Controller area network
    (CAN).
-   **Bosch CAN Specification** --- CAN protocol fundamentals and frame
    behavior.
-   **ISO/SAE 21434** --- Road vehicles --- Cybersecurity engineering.
-   **AUTOSAR** --- Automotive software architecture and
    security-related specifications.
-   **NIST** --- Cybersecurity guidance and security engineering
    resources.

------------------------------------------------------------------------

*This overview provides general technical background for the CANDICE
lab. Individual implementations may use narrower definitions, datasets,
or assumptions; those details should be documented in the corresponding
version README.*

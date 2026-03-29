# Data Dictionary: CICIDS2017 Network Intrusion Dataset

## 1. Dataset Overview: CICIDS2017

The dataset represents captured network traffic (pcap files) processed into CSV format using CICFlowMeter. It contains bidirectional network flows with various statistical features.


- Data collection spans 5 days
- Total Features: 78 independent variables + 1 target variable (Label)
- Feature Types: * Numeric (78): 54 int64 (counts, durations, ports) and 24 float64 (means, standard deviations, rates).
- Categorical (1): Label (The attack classification).

## 2. Feature Definitions

### A. Flow Identification & Metadata
| Variable | Type | Unit | Description |
| :--- | :--- | :--- | :--- |
| `Destination Port` | int64 | Port Number | The destination port of the network flow (e.g., 80, 443, 21). |
| `Flow Duration` | int64 | Microseconds | Total duration of the flow from start to finish. |
| `Label` | object | Categorical | Target Variable: `BENIGN` or specific attack (e.g., DDoS, PortScan). |

### B. Traffic Volume & Packet Size
| Variable | Type | Unit | Description |
| :--- | :--- | :--- | :--- |
| `Total Fwd Packets` | int64 | Count | Total packets sent in the forward direction. |
| `Total Backward Packets` | int64 | Count | Total packets sent in the backward direction. |
| `Total Length of Fwd Packets` | int64 | Bytes | Total size of payload in the forward direction. |
| `Total Length of Bwd Packets` | int64 | Bytes | Total size of payload in the backward direction. |
| `Fwd/Bwd Packet Length Max` | int64 | Bytes | Maximum size of a packet in the flow. |
| `Fwd/Bwd Packet Length Min` | int64 | Bytes | Minimum size of a packet in the flow. |
| `Fwd/Bwd Packet Length Mean` | float64 | Bytes | Average size of packets in the flow. |
| `Fwd/Bwd Packet Length Std` | float64 | Bytes | Standard deviation of packet sizes. |

### C. Flow Timings (Inter-Arrival Times)
| Variable | Type | Unit | Description |
| :--- | :--- | :--- | :--- |
| `Flow IAT Mean/Std/Max/Min` | float/int | Microseconds | Statistical time between two packets sent in the flow. |
| `Fwd IAT Total` | int64 | Microseconds | Total time between two packets sent in the forward direction. |
| `Bwd IAT Total` | int64 | Microseconds | Total time between two packets sent in the backward direction. |

### D. TCP Flags & Header Information
| Variable | Type | Unit | Description |
| :--- | :--- | :--- | :--- |
| `Fwd PSH/URG Flags` | int64 | Binary/Count | Number of times PSH/URG flags were set in forward direction. |
| `FIN/SYN/RST/ACK Flag Count` | int64 | Count | Total count of packets with the respective TCP flag. |
| `Fwd/Bwd Header Length` | int64 | Bytes | Total bytes used for headers in the forward/backward direction. |
| `Init_Win_bytes_forward` | int64 | Bytes | The number of bytes sent in initial window in the forward direction. |

### E. Network Performance & Activity
| Variable | Type | Unit | Description |
| :--- | :--- | :--- | :--- |
| `Flow Bytes/s` | float64 | Bps | Throughput: (Total Bytes) / (Flow Duration). |
| `Flow Packets/s` | float64 | Pps | Packet rate: (Total Packets) / (Flow Duration). |
| `Active Mean/Std/Max/Min` | float64 | Microseconds | Time a flow was active before going idle. |
| `Idle Mean/Std/Max/Min` | float64 | Microseconds | Time a flow was idle before becoming active. |
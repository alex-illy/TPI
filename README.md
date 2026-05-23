# Thesis 
Name: *Modeling and Model Transformation for Verification of ROS 2 Systems using Eclipse and UPPAAL*

The aim of this thesis is to rely on the research side, yet the thesis was solved with both a proof-of-concept and a proof-by-demonstration approach. 

This repository stores the documentation for the Proof-by-Demonstration Approach, where a automated Pipeline based on Lukas's work is implemented in 3 stages to verify a ROS 2 Systems inside of UPPAAL with 2 transformations inside of Eclipse. 

To achieve the goal of this thesis, the ping-pong example from CBG_Executor is used. During the 3 stages of the Pipeline, the traces from the execution of the Ping-Pong example are taken as input, and transformed in a ready to verify model inside of UPPAAL. 


### Prerequisites and Application Versions 
  OS: Ubuntu 22.04 with Real Time Kernel --> more details in this [documentation](ROS%202%20Humble%20Real%20Time%20Kernel%20with%20Tracing.md)
  Eclipse EMF 2026-03 (with the default Acceleo and QVT-O installations)
  UPPAAL 5.0.0


### Pipeline Implementation Documentation 

- [Stage 1 Tracing and Python Scripts](Stage%201%20Tracing%20%20and%20Python%20Scripts.md)
- [Stage 2 QVT-o Transformation](Stage%202%20QVT-o%20Transformation.md)
- [Stage 3 Acceleo Transformation](Stage%203%20Acceleo%20Transformation.md)


Following the 3 stages step by step will output a ready to verify UPPAAL complaint model of the ping-pong example from CBG_Executor.  You need an Ubuntu 22.04 with RT enabled to achieve the same result. No real time kernel implies no kernel traces, whom are critical for this thesis. 

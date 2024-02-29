## Problem Formulation

$$
\begin{aligned}
% &\textbf{Problem 3} && \textbf{Reduced Robustified NMPC (R²NMPC)}\\ 
&\textbf{Problem} && \textbf{R²NMPC with}\\ 
&&&\textbf{Ellipsoidal Uncertainty Sets}\\
% &&& \textbf{with probabilistic constraints}\\
&  \underset{\boldsymbol{x}(.), \boldsymbol{u}(.)}{\min} & & 
\begin{aligned}
    \int^{T_p}_{\tau=0} & l(\boldsymbol{x}(\tau),\boldsymbol{u}(\tau)) \space  d\tau \\ 
 & + m(\boldsymbol{x}(T_p))
\end{aligned}
  \\ 
& \text{s. t.} & & \boldsymbol{x}_{0} \leq \boldsymbol{x}(0) \leq \boldsymbol{x}_{0} \text {, }  \\
% & & &  \text { --- vehicle dynamics ---} \\
& & & \dot{\boldsymbol{x}}(t) = f(\boldsymbol{x}(t),\boldsymbol{u}(t), \boldsymbol{O}_{n_x}) \text{, } & t \in[0, T_p), \\
& & & 
% h(\boldsymbol{x}(t), \boldsymbol{u}(t)) 
\begin{aligned}
    \boldsymbol{h}&(\boldsymbol{x}(t), \boldsymbol{u}(t))  \\ & + \boldsymbol{h_{\text{back}}}(\boldsymbol{x}(t), \boldsymbol{u}(t), \boldsymbol{\Sigma}(t)) \leq \bar{\boldsymbol{h}}, 
   \end{aligned}
& t \in[0, T_p), \\
& & & 
\begin{aligned}
    \boldsymbol{h}^{\mathrm{e}}&(\boldsymbol{x}(T_p)) \\
    & + \boldsymbol{h_{\text{back}}}^{\mathrm{e}}(\boldsymbol{x}(T_p), \boldsymbol{\Sigma}(T_p))
\leq \bar{\boldsymbol{h}}^{\mathrm{e}}
\end{aligned}\\
\end{aligned}
$$

The main idea of the R²NMPC can be summarized in the following 3 differences to the original Robustified NMPC:
- We carry out the ellipsoidal uncertainty propagation outside the OCP. This simplifies the problem by eliminating optimization variables related to the propagation of ellipsoidal uncertainty sets $\boldsymbol{\Sigma}$ with $\Phi$, resulting in a reduced formulation that matches the dimension of the nominal NMPC, containing $n_x$ variables.

- We approximate the ellipsoidal uncertainty sets propagation $\dot{\boldsymbol{\Sigma}}$ using the sensitivities $\boldsymbol{A}$ and $\boldsymbol{B}$ (Eq.\ref{jacobians}) of the system dynamics obtained from the last R²NMPC solution at each shooting node with the discrete-time Lyapunov Dynamics, as in Eq.\ref{eq:discrete_Sigma_dynamics}. 
This solution takes advantage of the feedback-loop and assumes that there is no big deviations in the dynamics between two consecutive R²NMPC solutions. 
$$
        \boldsymbol{\Sigma}(t+1) = \boldsymbol{A}(t)\boldsymbol{\Sigma}(t)\boldsymbol{A}^T(t)+\boldsymbol{B}(t)\boldsymbol{W}(t)\boldsymbol{B}^T(t)
    $$

- We approximate the back-off terms $\boldsymbol{h_{\text{back}}}$ at each shooting node using the gradients $\nabla_{\boldsymbol{x}} \boldsymbol{h}(\boldsymbol{x}(t), \boldsymbol{u}(t))$ computed from the last R²NMPC solution.

## Cost function
The stage cost is defined as a nonlinear least square function: $l(\boldsymbol{x}, \boldsymbol{u})=\frac{1}{2}\|\boldsymbol{y}(\boldsymbol{x},\boldsymbol{u})-\boldsymbol{y}_{\mathrm{ref}}\|_W^2$. Similarly, the terminal cost is formulated as $m(\boldsymbol{x})=\frac{1}{2}\|\boldsymbol{y}^e(\boldsymbol{x})-\boldsymbol{y}^e_{\mathrm{ref}}\|_{W^e}^2$. Here, $W$ and $W_e$ represent the weighting matrices for the stage and terminal costs, respectively. $W$ is computed as $W = \text{diag}(Q,R)$, where $Q$ and $R$ are matrices for states and inputs weighting, while $W_e$ is defined as $W_e = Q_e$. The cost terms are defined as follows: 
$$
  \begin{aligned}
\boldsymbol{y}(\boldsymbol{x},\boldsymbol{u}) &= [x_{\text{pos}},\space y_{\text{pos}},\space \psi,\space v_{\text{lon}},\space  j, \space \omega_f] \\
\boldsymbol{y}_{\text{ref}} &= [x_{\text{pos,ref}},\space y_{\text{pos,ref}},\space \psi_{\text{ref}},\space v_{\text{ref}},\space  0,\space 0] \\
\boldsymbol{y}^e_{\text{ref}} &= [x_{\text{pos,ref}}^e,\space y_{\text{pos,ref}}^e,\space \psi^e_{\text{ref}},\space v^e_{\text{ref}}]\\
\end{aligned}  
$$
Moreover, we employ two slack variables, $L1$ for a linear- and $L2$ for a quadratic constraint violation penalization term, relaxing the constraints and helping the solver find a solution.

To determine appropriate matrices $Q$ and $Q_e$ and $R$, we employ Multi-Objective Bayesian Optimization:
$
    Q = Q_e =  \text{diag} ( 
        2.8, \space 
        2.8, \space 
        0.4, \space 
        0.2
    )
$
and 
$
    R= \text{diag} ( 
        38.1, \space 
        101.4
    )
$.

## Bibliography
[1] Zarrouki, B., Nunes, J., & Betz, J. (2023). R $^ 2$ NMPC: A Real-Time Reduced Robustified Nonlinear Model Predictive Control with Ellipsoidal Uncertainty Sets for Autonomous Vehicle Motion Control. https://arxiv.org/abs/2311.06420
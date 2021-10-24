---
classes: wide
title: Darcy-Weisbach Sürtünme Katsayısının Hesabı
published: false
mathjax: true
categories:
  - Python
  - Hidrolik
tags:
  - Darcy-Weisbach
---
Sürtünme katsayısı, laminer, laminer ve türbülanslı akımlar arasındaki geçiş, tamamen türbülanslı ve serbest yüzeyli akım koşulları için hesaplanabilir.

**Laminer akım:**
Laminer akım koşullarında reynolds katsayısı 2000'den küçüktür. $$f$$ sürtünme katsayısı, $$\mathrm{Re}$$ reynolds katsayısı olmak üzere, \\[f = \frac{64}{\mathrm{Re}} \\] formülü ile hesaplanabilir. 

**Geçiş akımı:**
Geçiş akımı ne tam olarak laminer ne de türbülanslıdır ve reynolds katsayısının 2000 ile 4000 arasındaki değerleri için oluşur. Bu bölgede sürtünme katsayısının hesabı için yeterli bilgi bulunmamaktadır. Ancak Swamee (1993) bu bölge için aşağıdaki formülü önermiştir. ($$D_{\mathrm{e}}= 4 \frac{Alan}{Çevre}$$)
\\[f=\left[ \left( \frac{64}{\mathrm{Re}} \right) ^{8} +9.5 \left( \log \left( \frac { \varepsilon} {3.7 D_\mathrm{e}} + \frac {5.74} {\mathrm{Re}^{0.9}} \right) - \left( \frac{2500}{\mathrm{Re}} \right)^{6} \right) ^{-16} \right] ^{0.125}\\]
**Pürüzsüz borularda tam türbülanslı akım:** Blasius korelasyonu bu bölge için sürtünme katsayısını hesaplamakta kullanılan en basit eşitliktir. Reynolds sayısnın 100000'den büyük olduğu değerler için geçerli değildir. \\[f = 0.316 \mathrm{Re}^{-{1 \over 4}}\\] 
**Pürüzlü borularda tam türbülanslı akım:**Reynols katsayısının 4000'den büyük olduğu bu bölgede sürtünme katsayısı Colebrook eşitliği ile hesaplanabilir. \\[\frac{1}{\sqrt{f}}= -2 \log_{10} \left( \frac { \varepsilon} {3.7 D_\mathrm{e}} + \frac {2.51} {\mathrm{Re} \sqrt{f}} \right)\\]Colebrook eşitliği kapalı yapısı nedeniyle sayısal olarak çözülebilmektedir. Bu eşitlğin yaklaşık çözümleri olan Haaland ve Swamee–Jain eşitlikleri aşağıda verilmiştir. 

Halland eşitliği, \\[\frac{1}{\sqrt {f}} = -1.8 \log_{10} \left[ \left( \frac{\varepsilon/D_{\mathrm{e}}}{3.7} \right)^{1.11} + \frac{6.9}{\mathrm{Re}} \right]\\] 
Swamee–Jain eşitliği,\\[f = 0.25 \left[\log_{10} \left(\frac{\varepsilon}{3.7D_{\mathrm{e}}} + \frac{5.74}{\mathrm{Re}^{0.9}}\right)\right]^{-2}\\]
**Serbest yüzeyli akım:**
$$R_{\mathrm{h}}$$ hidrolik yarıçap olmak üzere, sürtünme katsayısı, Colebrook eşitliğinin serbest yüzeyli akımlar için geçerli olan aşağıdaki biçimiyle hesaplanabilir. \\[\frac{1}{\sqrt{f}} = -2 \log_{10} \left(\frac{\varepsilon}{12R_\mathrm{h}} + \frac{2.51}{\mathrm{Re}\sqrt{f}}\right)\\]

Yukardaki basınçlı akım koşulları için Darcy-Weisbach sürtünme katsayısını hesaplayan DarcyWeisbachFrictionFactor python sınıfını kodunu aşağıda bulabilirsiniz.

{% highlight python %}

from math import pow, pi, log, log10, sqrt
from enum import Enum
from scipy.optimize import fsolve


class TransitionFlowSolveMethod (Enum):
    cubic_interpolation_moody_diagram = 1
    swamee_jain_approximation = 2


class TurbulentFlowSolveMethod (Enum):
    colebrook_white_equation = 1
    swamee_jain_approximation = 2
    halland_approximation = 3


class DarcyWeisbachFrictionFactor:
    def __init__(self, Q, D, A=0.00, P=0.00, e=0.0000457, v=0.000001002,
                 solve_transition_flow_for=TransitionFlowSolveMethod.swamee_jain_approximation,
                 solve_turbulent_flow_for=TurbulentFlowSolveMethod.colebrook_white_equation):
        """
        Q: Flow discharge [m³/sn]
        D: Diameter of flow section [m]
        A: Flow cross-sectional area [m²]
        P: Wetted perimeter [m]
        e: Roughness height [m] -> e = 4.57e-5 m (Stainless steel)
        v: Kinematic viscosity [m²/s)] -> v = dynamic viscosity/density = [kg.m/sn] / [kg/m³]
           For T=20° v = 1.002e-6
        """
        self.Q = Q
        self.e = e
        self.v = v
        self.f = 0.0
        if A == 0.0 or P == 0.0:
            self.A = pi * pow(D, 2.0) / 4.0
            self.P = pi * D
        else:
            self.A = A
            self.P = P
        self.De = 4 * self.A / self.P
        self.Re = self.Q * self.De / (self.v * self.A)
        self.smooth_pipe_turbulent_f = 0.316 / pow(self.Re, 0.25)
        self.completely_turbulent_f = 1.0 / pow(1.14 + 2.00 * log10(self.De / self.e), 2.00)

        if self.Re <= 2000.0:
            self.f = 64 / self.Re
        elif self.Re >= 4000.0:
            if solve_turbulent_flow_for == TurbulentFlowSolveMethod.swamee_jain_approximation:
                self.f = 1.325 / pow(log(self.e / (3.70 * self.De) + 5.74 / (pow(self.Re, 0.9))), 2.0)
            elif solve_turbulent_flow_for == TurbulentFlowSolveMethod.halland_approximation:
                self.f = 0.3086 / pow(log10(pow(self.e / (3.70 * self.De), 1.11) + 6.9 / self.Re), 2.0)
            elif solve_turbulent_flow_for == TurbulentFlowSolveMethod.colebrook_white_equation:
                initial_guess = 1.325 / pow(log(self.e / (3.70 * self.De) + 5.74 / (pow(self.Re, 0.9))), 2.0)
                self.f = fsolve(lambda f: 1.00 / sqrt(f) + 2.0 * log10(self.e / self.De / 3.70 + 2.51 /
                                                                       (self.Re * sqrt(f))), initial_guess)[0]
        else:
            if solve_transition_flow_for == TransitionFlowSolveMethod.swamee_jain_approximation:
                a1 = pow(64 / self.Re, 8.0)
                a2 = log(e / (3.70 * self.De) + 5.74 / (pow(self.Re, 0.9)))
                a3 = pow(2500 / self.Re, 6.0)
                a4 = pow(a2 - a3, 16.0)
                self.f = pow(a1 + 9.50 / a4, 0.125)
            elif solve_transition_flow_for == TransitionFlowSolveMethod.cubic_interpolation_moody_diagram:
                # from epanet program methodology page 6-23
                r = self.Re / 2000.0
                y2 = e / (3.7 * self.De) + 5.74 / pow(4000.0, 0.90)
                y3 = -0.8685889638 * log(y2)
                fa = 1.0 / pow(y3, 2.0)
                fb = fa * (2.0 - 0.00514214965799 / (y2 * y3))
                x1 = 7.0 * fa - fb
                x2 = 0.128 - 17.0 * fa + 2.5 * fb
                x3 = -0.128 + 13.0 * fa - 2.0 * fb
                x4 = r * (0.032 - 3.0 * fa + 0.5 * fb)
                self.f = x1 + r * (x2 + r * (x3 + x4))

    @property
    def kinematic_viscosity_for_water(self, t: float) -> float:
        return 1.792e-6 / (1 + pow(t / 25, 1.165))

{% endhighlight %}



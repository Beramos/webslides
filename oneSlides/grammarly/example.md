Neural differential eqautions are differential equations where the right-hand side of the equations (@eq:neuralODE) are replaced by an ANN. Instead of learning time series differential equations are being fitted to the data as a compact representation of the underlying mechanisms [@Rackauckas2019].

$$\frac{\mathrm{d}R}{\mathrm{d}t} = ANN(c_\mathrm{NaCl}, \,J, \, v)$$ {#eq:neuralODE}

with $R$ the membrane resistance, $t$ time, $c_\mathrm{NaCl}$ the concentration of NaCl, $J$ the current density and $v$ the crossflow velocity. The data of @Korngold1970 was split in two parts. The first part is used to train the model while the second part of the data was use to demonstrate predictive capabilities of this model (model validation). 
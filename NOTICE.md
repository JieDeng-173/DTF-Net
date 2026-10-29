# Notices and provenance

This partial release includes the supplied DTF-Net fusion implementation and
new packaging, cached-input inference helpers, documentation and tests.
`dtf_net/model.py` is copied without modification from the supplied project.
Its existing fusion-mode switches are retained for compatibility; complete
ablation/training workflows are not released.

The repository layout was informed by VascHAT:
https://github.com/BoxiangLiu-111/VascHAT
No VascHAT/HAT source code is included or required.

The complete research pipeline uses external methods:
- Diffeomorphic Template: https://github.com/donglao/DiffeomorphicTemplate
  Lao et al., Diffeomorphic Template Registration for Atmospheric Turbulence
  Mitigation, CVPR 2024.
- TMT: https://xg416.github.io/TMT/
  Zhang et al., Imaging through the Atmosphere using Turbulence Mitigation
  Transformer, IEEE Transactions on Computational Imaging, 2024.

These external implementations and their checkpoints are not redistributed.
Their respective licenses and notices apply independently.
The included code is released under Apache-2.0 as authorized by the project owner.

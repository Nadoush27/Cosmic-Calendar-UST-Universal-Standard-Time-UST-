# Cosmic Calendar – Universal Standard Time (UST)

**Complete reference implementation – November 2025**  
Author: Nadia Sahraoui  

Fully working, decentralised, quantum-ready time standard for Earth • Moon • Mars.

### Features
- 13-phase lunar synodic framework (29.530587971 days)
- Real relativistic corrections (NASA/ESA values)
- Zero server dependency – runs on any spacecraft or base
- MIT license (code)

### Quick start
```python
from cosmic_ust import ust_string, ust_now
print(ust_string(*ust_now("Moon")))

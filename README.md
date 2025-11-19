# Cosmic-Calendar-UST-Universal-Standard-Time-UST-
Universal Standard Time (UST) – Complete 13-phase lunar time standard for Earth, Moon, Mars. Reference implementation + paper by Nadia Sahraoui
"""
Universal Standard Time (UST) - Cosmic Calendar Reference Implementation
Author: Nadia Sahraoui
Version: 2.0  (Updated per Cosmic Calendar V2)
License: MIT

This module implements the UST algorithm based on:
- The 13-phase lunar synodic framework
- Relativistic gravitational offsets (microseconds/day)
- Quantum-aligned timekeeping logic (conceptual foundation)

GitHub (to be added when you provide it): https://github.com/______

Epoch:
Astronomical New Moon of 06 January 2000 at 09:16 UTC.
This is the standard synodic reference for all UST calculations.
"""

import datetime
from datetime import timezone
from math import floor

# =============================================================
# COSMIC CONSTANTS
# =============================================================

SYNODIC_MONTH_DAYS = 29.5305879710734      # mean synodic lunar month
PHASES_PER_CYCLE = 13
DAYS_PER_PHASE = SYNODIC_MONTH_DAYS / PHASES_PER_CYCLE   # ≈ 2.271584459 days

# Relativistic offsets (microseconds/day) — conceptual baseline for V2
RELATIVISTIC_OFFSET = {
    'Earth': 0.0,     # Reference frame
    'Moon': 56.5,     # Lunar LTC average relativistic correction
    'Mars': -43.2     # Martian GR + orbital approximation
}

# Astronomical New Moon reference (IAU standard epoch)
UST_EPOCH = datetime.datetime(2000, 1, 6, 9, 16, 0, tzinfo=timezone.utc)


# =============================================================
# CORE FUNCTIONS
# =============================================================

def ust_now(location='Earth'):
    """
    Returns the current UTC timestamp.
    UST adjustment is applied only when functions consume this timestamp.
    """
    return datetime.datetime.now(timezone.utc), location


def days_since_epoch(dt):
    """
    Returns number of Earth days elapsed since UST epoch.
    """
    delta = dt - UST_EPOCH
    return delta.total_seconds() / 86400.0


def apply_relativistic_adjustment(days, location='Earth'):
    """
    Applies relativistic correction based on microseconds drift per day.
    Cleaned and physically consistent for Version 2.

    corrected_days = days * (1 + OFFSET_microseconds/day / 86400)

    Returns adjusted day-count.
    """
    offset = RELATIVISTIC_OFFSET.get(location, 0.0)
    correction_factor = 1 + (offset * 1e-6) / 86400.0
    return days * correction_factor


def ust_phase_and_cycle(dt, location='Earth'):
    """
    Computes:
    - UST cycle number
    - UST phase (1–13)
    - Phase progress (0–100 %)
    """
    days = days_since_epoch(dt)
    corrected_days = apply_relativistic_adjustment(days, location)

    # Position inside synodic cycle
    days_into_cycle = corrected_days % SYNODIC_MONTH_DAYS

    phase_float = days_into_cycle / DAYS_PER_PHASE
    phase_number = floor(phase_float) + 1

    # Clamp to [1,13]
    if phase_number > 13:
        phase_number = 13

    phase_progress = (phase_float % 1.0) * 100
    cycle_number = floor(corrected_days / SYNODIC_MONTH_DAYS)

    return phase_number, phase_progress, cycle_number


def ust_string(dt, location='Earth'):
    """
    Human-readable output of the UST state.
    """
    phase, progress, cycle = ust_phase_and_cycle(dt, location)
    return (
        f"UST Cycle {cycle} • Phase {phase}/13 "
        f"({progress:.1f}%) • "
        f"{dt.strftime('%Y-%m-%d %H:%M:%S')} ({location})"
    )


# =============================================================
# EXAMPLES
# =============================================================

if __name__ == "__main__":
    now_utc, loc = ust_now("Moon")
    print(ust_string(now_utc, loc))

    # Test vector — New Moon 1 Jan 2025
    test_date = datetime.datetime(2025, 1, 1, 0, 0, 0, tzinfo=timezone.utc)
    print("Test 2025 New Moon:", ust_string(test_date, "Earth"))

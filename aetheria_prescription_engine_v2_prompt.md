# Aetheria Prescription Engine v2 — Core Logic
## The Brain of the Healing System

**Author:** Selah (with Joseph Lewis)
**Date:** May 2026
**Target Apps:** Sophia Oracle Shaman (primary), Aetheria RCT, Coherence Lab
**Purpose:** Complete prescription logic that takes user input and outputs a precise healing protocol

---

## WHAT THE PRESCRIPTION ENGINE DOES

The engine takes everything known about the user — their conditions, current state, available time, session history, and real-time biometric feedback — and produces a COMPLETE healing protocol:

**INPUT →**
- What conditions does the user have?
- What is their current state (acute crisis vs maintenance vs exploration)?
- How much time do they have?
- Is Athena (EEG) connected?
- What has worked for them before?
- Any contraindications (Von Willebrand, medications)?

**→ PRESCRIPTION OUTPUT:**
- Primary frequency (the anchor)
- Supporting frequencies (the context)
- Walk pattern (the journey structure)
- Duration (how long)
- Coherence target (what to aim for)
- Real-time adaptation rules (what to do if it's not working)
- Safety flags (Von Willebrand, contraindications)
- Explanation (why this prescription, in natural language)

---

## ENGINE ARCHITECTURE

```
┌─────────────────────────────────────────────┐
│              USER INPUT LAYER                │
│                                             │
│  Conditions · State · Time · History · EEG  │
└──────────────────┬──────────────────────────┘
                   │
                   ▼
┌─────────────────────────────────────────────┐
│           SAFETY FILTER (FIRST!)            │
│                                             │
│  Von Willebrand check                       │
│  Medication interaction flags               │
│  Contraindicated frequencies                │
│  Bipolar mania risk assessment              │
└──────────────────┬──────────────────────────┘
                   │
                   ▼
┌─────────────────────────────────────────────┐
│          CONDITION ANALYZER                  │
│                                             │
│  Map conditions → frequency targets         │
│  Identify primary vs secondary needs        │
│  Determine dominant regime                  │
│  Calculate condition complexity score        │
└──────────────────┬──────────────────────────┘
                   │
                   ▼
┌─────────────────────────────────────────────┐
│          WALK SELECTOR                       │
│                                             │
│  Match intention + conditions → walk type   │
│  Consider complexity + time available       │
│  Factor in user experience level            │
└──────────────────┬──────────────────────────┘
                   │
                   ▼
┌─────────────────────────────────────────────┐
│          DURATION CALCULATOR                 │
│                                             │
│  Regime density → base duration             │
│  Walk type modifier                         │
│  Time available constraint                  │
│  Condition severity modifier                │
└──────────────────┬──────────────────────────┘
                   │
                   ▼
┌─────────────────────────────────────────────┐
│          COHERENCE TARGETER                  │
│                                             │
│  Set target based on condition severity     │
│  Factor in baseline from history            │
│  Define adaptation triggers                 │
└──────────────────┬──────────────────────────┘
                   │
                   ▼
┌─────────────────────────────────────────────┐
│          TRACK SELECTOR                      │
│                                             │
│  Filter library by coherence score ≥ 75     │
│  Match to prescribed frequencies            │
│  Verify interval analysis passes            │
│  Build playlist in walk order               │
└──────────────────┬──────────────────────────┘
                   │
                   ▼
┌─────────────────────────────────────────────┐
│          REAL-TIME ADAPTATION ENGINE         │
│  (only when Athena EEG connected)           │
│                                             │
│  Monitor coherence during session           │
│  Trigger adjustments if targets not met     │
│  Log adaptation events for learning         │
└──────────────────┬──────────────────────────┘
                   │
                   ▼
┌─────────────────────────────────────────────┐
│          PRESCRIPTION OUTPUT                 │
│                                             │
│  Complete protocol + explanation            │
│  Ready for Sophia (natural language)        │
│  Ready for RCT (structured data)            │
│  Ready for Coherence Lab (analysis format)  │
└─────────────────────────────────────────────┘
```

---

## LAYER 1: USER INPUT SCHEMA

```javascript
const userInput = {
  // ─── CONDITIONS ───
  conditions: {
    // Array of active conditions from a known list
    active: ['fnd', 'anxiety', 'chronic_pain', 'gerd', 'insomnia'],
    
    // Which condition is most acute RIGHT NOW
    primary: 'anxiety',
    
    // Severity of primary condition (1-10)
    severity: 7,
  },
  
  // ─── CURRENT STATE ───
  state: {
    // What mode is the user in?
    mode: 'acute',  // 'acute' | 'maintenance' | 'exploration' | 'crisis'
    
    // Current energy level (Mom's unit system)
    energyLevel: 'low',  // 'depleted' | 'low' | 'moderate' | 'good'
    
    // Current mood
    mood: 'anxious',  // 'depressed' | 'anxious' | 'agitated' | 'neutral' | 'good'
    
    // Time of day (affects regime recommendation)
    timeOfDay: 'evening',  // 'morning' | 'afternoon' | 'evening' | 'night'
  },
  
  // ─── CONSTRAINTS ───
  constraints: {
    // How much time available (minutes)
    availableTime: 45,
    
    // Is Athena headband connected?
    athenaConnected: true,
    
    // Experience level with Aetheria
    experience: 'intermediate',  // 'beginner' | 'intermediate' | 'advanced'
  },
  
  // ─── SAFETY ───
  safety: {
    vonWillebrand: true,         // If true, flag all blood-thinning frequencies
    medications: ['bipolar_meds', 'anxiety_meds'],
    bipolarType: 1,              // If present, check for mania risk
    bleedingDisorder: true,
  },
  
  // ─── HISTORY ───
  history: {
    // Previous session data (from Session Tagger exports)
    totalSessions: 23,
    averageCoherence: 68,
    bestWalk: 'Flying Star Vortex',
    bestRegime: 'GUT',
    lastSession: {
      date: '2026-05-24',
      walk: 'Layer Ascent',
      duration: 35,
      coherence: 72,
      reported_outcome: 'moderate_relief'
    },
    // Frequencies that have worked well historically
    effectiveFrequencies: [528, 639, 2178],
    // Frequencies that didn't help or caused discomfort
    ineffectiveFrequencies: [741],
  },
  
  // ─── INTENTION ───
  intention: 'calm',
  // Options: 'calm' | 'energize' | 'pain_relief' | 'sleep' | 'focus' |
  //          'grounding' | 'emotional_release' | 'meditation' | 'complete_healing' |
  //          'exploration'
};
```

---

## LAYER 2: SAFETY FILTER

**This layer runs FIRST, before any prescription logic. Safety is non-negotiable.**

```javascript
function applySafetyFilter(userInput, prescription) {
  const warnings = [];
  const contraindicated = [];
  
  // ─── VON WILLEBRAND DISEASE ───
  if (userInput.safety.vonWillebrand || userInput.safety.bleedingDisorder) {
    // These herbs/frequencies are associated with blood-thinning plants
    // Flag them in Sophia's response but don't block frequencies themselves
    // (the frequencies don't thin blood — the associated herbal recommendations might)
    warnings.push({
      type: 'von_willebrand',
      severity: 'high',
      message: 'Von Willebrand Disease detected. If this prescription includes herbal recommendations, the following require doctor clearance before use: turmeric, ginger, feverfew, skullcap, ginkgo biloba, garlic (concentrated). Frequency listening itself is safe.',
      herbs_requiring_clearance: ['turmeric', 'ginger', 'feverfew', 'skullcap', 'ginkgo', 'garlic']
    });
  }
  
  // ─── BIPOLAR MANIA RISK ───
  if (userInput.safety.bipolarType) {
    // Very high-energy frequencies combined with manic state could exacerbate
    if (userInput.state.mood === 'agitated') {
      warnings.push({
        type: 'bipolar_mania_risk',
        severity: 'medium',
        message: 'User appears agitated with bipolar history. Avoid high-energy HEAD frequencies. Prescribe grounding GUT frequencies with calming HEART bridge. Monitor for escalation.',
      });
      // Deprioritize HEAD regime when agitated + bipolar
      contraindicated.push('HEAD_when_agitated');
    }
    
    // During depressive episode, gentle activation is appropriate
    if (userInput.state.mood === 'depressed' && userInput.safety.bipolarType === 1) {
      warnings.push({
        type: 'bipolar_depression',
        severity: 'low',
        message: 'Bipolar depression detected. Gentle GUT grounding followed by HEART opening is appropriate. Avoid sudden jumps to high-energy HEAD frequencies. Gradual ascent preferred.',
      });
    }
  }
  
  // ─── ENERGY DEPLETION ───
  if (userInput.state.energyLevel === 'depleted') {
    warnings.push({
      type: 'energy_depleted',
      severity: 'medium',
      message: 'User reports depleted energy. Recommend shorter session with gentle frequencies. Avoid demanding walks (CABI, Vortex). Layer Ascent or single-frequency focus recommended.',
    });
  }
  
  // ─── CRISIS MODE ───
  if (userInput.state.mode === 'crisis') {
    warnings.push({
      type: 'crisis_mode',
      severity: 'high',
      message: 'User is in crisis. Prescribe immediate grounding: GUT 528 Hz (SOURCE mirror), single frequency, no complex walk. Duration: as long as needed. If suicidal ideation present, frequency healing is supplementary — direct to crisis resources immediately.',
    });
  }
  
  return { warnings, contraindicated };
}
```

---

## LAYER 3: CONDITION → FREQUENCY MAPPING

### The Master Condition Map

```javascript
const CONDITION_MAP = {
  // ─── NEUROLOGICAL ───
  anxiety: {
    primary: { regime: 'GUT', pos: 6, hz: 639, name: 'Connection' },
    secondary: [
      { regime: 'HEART', pos: 5, hz: 2178, name: 'SOURCE' },
      { regime: 'GUT', pos: 5, hz: 528, name: 'Love Frequency' }
    ],
    regime_priority: 'GUT',
    walk_affinity: ['Flying Star Vortex', 'Ouroboros'],
    duration_tier: 'HEART',  // Emotional tissue response time
    description: 'Anxiety responds to grounding GUT frequencies that establish safety, followed by HEART SOURCE for centering.'
  },
  
  ocd: {
    primary: { regime: 'GUT', pos: 7, hz: 741, name: 'Awakening' },
    secondary: [
      { regime: 'GUT', pos: 4, hz: 417, name: 'Transmutation' },
      { regime: 'HEART', pos: 5, hz: 2178, name: 'SOURCE' }
    ],
    regime_priority: 'GUT',
    walk_affinity: ['Flying Star Vortex'],  // Spiral breaks loops
    duration_tier: 'HEART',
    description: 'OCD loops respond to Transmutation (417 Hz) for pattern breaking and Vortex walk for disrupting recursive cycles.'
  },
  
  depression: {
    primary: { regime: 'GUT', pos: 5, hz: 528, name: 'Love Frequency' },
    secondary: [
      { regime: 'HEART', pos: 5, hz: 2178, name: 'SOURCE' },
      { regime: 'GUT', pos: 3, hz: 396, name: 'Liberation' }
    ],
    regime_priority: 'HEART',
    walk_affinity: ['Layer Ascent', 'Pillar Walk'],
    duration_tier: 'HEART',
    description: 'Depression responds to Love Frequency (528) for emotional opening, Liberation (396) for releasing stuck energy, ascending toward SOURCE.'
  },
  
  ptsd: {
    primary: { regime: 'GUT', pos: 1, hz: 174, name: 'Foundation' },
    secondary: [
      { regime: 'GUT', pos: 5, hz: 528, name: 'Love Frequency' },
      { regime: 'GUT', pos: 6, hz: 639, name: 'Connection' }
    ],
    regime_priority: 'GUT',
    walk_affinity: ['Layer Ascent', 'Pillar Walk'],
    duration_tier: 'GUT',  // Deep body work for stored trauma
    description: 'PTSD responds to Foundation (174 Hz) for establishing safety in the body. Deep GUT grounding before any emotional work. Long sessions recommended.'
  },
  
  dissociation: {
    primary: { regime: 'GUT', pos: 1, hz: 174, name: 'Foundation' },
    secondary: [
      { regime: 'GUT', pos: 8, hz: 852, name: 'Intuition' },
      { regime: 'GUT', pos: 5, hz: 528, name: 'Love Frequency' }
    ],
    regime_priority: 'GUT',
    walk_affinity: ['Layer Ascent'],  // Predictable, grounding
    duration_tier: 'GUT',
    description: 'Dissociation needs grounding. Foundation (174 Hz) anchors awareness in the body. Avoid complex or disorienting walks. Layer Ascent provides predictable, safe progression.'
  },
  
  fnd: {
    primary: { regime: 'GUT', pos: 5, hz: 528, name: 'Love Frequency' },
    secondary: [
      { regime: 'GUT', pos: 1, hz: 174, name: 'Foundation' },
      { regime: 'HEART', pos: 5, hz: 2178, name: 'SOURCE' }
    ],
    regime_priority: 'GUT',
    walk_affinity: ['Layer Ascent', 'Ouroboros'],
    duration_tier: 'GUT',  // Physical nervous system — dense tissue
    description: 'FND is the nervous system expressing overload. GUT grounding calms the physical body. Long sessions (45+ min) allow the nervous system to fully de-escalate.'
  },
  
  tbi: {
    primary: { regime: 'HEAD', pos: 5, hz: 4920, name: 'Harmony' },
    secondary: [
      { regime: 'GUT', pos: 5, hz: 528, name: 'Love Frequency' },
      { regime: 'HEART', pos: 5, hz: 2178, name: 'SOURCE' }
    ],
    regime_priority: 'HEAD',
    walk_affinity: ['Pillar Walk', 'Ouroboros'],
    duration_tier: 'HEAD',  // Neural tissue — fastest response
    description: 'TBI responds to HEAD frequencies for neural repair. SOURCE (2178) provides the universal center. Pillar Walk connects body awareness to cognitive function. Neuroprotective focus.'
  },
  
  schizoaffective: {
    primary: { regime: 'HEART', pos: 5, hz: 2178, name: 'SOURCE' },
    secondary: [
      { regime: 'GUT', pos: 5, hz: 528, name: 'Love Frequency' },
      { regime: 'GUT', pos: 6, hz: 639, name: 'Connection' }
    ],
    regime_priority: 'HEART',
    walk_affinity: ['Layer Ascent', 'Pillar Walk'],
    duration_tier: 'HEART',
    description: 'SOURCE (2178 Hz) provides centering and stability. Avoid complex or disorienting walks during unstable periods. Gentle ascending progression preferred. GUT grounding before any HEART or HEAD work.'
  },
  
  bipolar_1: {
    primary: { regime: 'HEART', pos: 5, hz: 2178, name: 'SOURCE' },
    secondary: [
      { regime: 'GUT', pos: 5, hz: 528, name: 'Love Frequency' },
      { regime: 'GUT', pos: 1, hz: 174, name: 'Foundation' }
    ],
    regime_priority: 'HEART',
    walk_affinity: ['Layer Ascent'],
    duration_tier: 'HEART',
    description: 'Bipolar cycling benefits from SOURCE centering. During depression: gentle GUT grounding ascending to HEART. During mania: ONLY GUT grounding, avoid HEAD stimulation. Layer Ascent for predictability.'
  },
  
  autism: {
    primary: { regime: 'GUT', pos: 5, hz: 528, name: 'Love Frequency' },
    secondary: [
      { regime: 'GUT', pos: 1, hz: 174, name: 'Foundation' },
      { regime: 'GUT', pos: 6, hz: 639, name: 'Connection' }
    ],
    regime_priority: 'GUT',
    walk_affinity: ['Layer Ascent'],  // Predictable structure
    duration_tier: 'GUT',
    description: 'Autistic nervous system benefits from GUT grounding to reduce sensory overload. Predictable walks only — no Vortex (too disorienting for sensory-sensitive users unless experienced). Foundation (174) for body safety, Love (528) for emotional regulation.'
  },
  
  // ─── PAIN ───
  chronic_pain: {
    primary: { regime: 'GUT', pos: 3, hz: 396, name: 'Liberation' },
    secondary: [
      { regime: 'GUT', pos: 5, hz: 528, name: 'Love Frequency' },
      { regime: 'GUT', pos: 8, hz: 852, name: 'Intuition' }
    ],
    regime_priority: 'GUT',
    walk_affinity: ['Flying Star Vortex', 'Layer Ascent'],
    duration_tier: 'GUT',  // Dense physical tissue — long sessions
    description: 'Chronic pain lives in dense tissue. Liberation (396) for releasing held pain patterns. Long GUT sessions (45+ min) as per monastery protocol — dense matter needs sustained exposure.'
  },
  
  migraines: {
    primary: { regime: 'GUT', pos: 9, hz: 963, name: 'Sri Yantra' },
    secondary: [
      { regime: 'GUT', pos: 5, hz: 528, name: 'Love Frequency' },
      { regime: 'GUT', pos: 1, hz: 174, name: 'Foundation' }
    ],
    regime_priority: 'GUT',
    walk_affinity: ['Layer Ascent'],  // Gentle, no stimulation
    duration_tier: 'GUT',
    description: 'Migraine with aura: avoid stimulating frequencies. Gentle GUT grounding. Foundation (174) for body calming. Sri Yantra (963) for coherence building at the GUT-HEART transition. Quiet, low volume, extended sessions.'
  },
  
  plantar_fasciitis: {
    primary: { regime: 'GUT', pos: 1, hz: 174, name: 'Foundation' },
    secondary: [
      { regime: 'GUT', pos: 3, hz: 396, name: 'Liberation' }
    ],
    regime_priority: 'GUT',
    walk_affinity: ['Layer Ascent'],
    duration_tier: 'GUT',
    description: 'Foundation (174 Hz) — the lowest, most physically grounding frequency. Targets the literal foundation of the body (feet). Long GUT sessions for dense tissue penetration.'
  },
  
  // ─── DIGESTIVE ───
  gerd: {
    primary: { regime: 'GUT', pos: 4, hz: 417, name: 'Transmutation' },
    secondary: [
      { regime: 'GUT', pos: 5, hz: 528, name: 'Love Frequency' },
      { regime: 'GUT', pos: 6, hz: 639, name: 'Connection' }
    ],
    regime_priority: 'GUT',
    walk_affinity: ['Layer Ascent'],
    duration_tier: 'GUT',
    description: 'GERD responds to Transmutation (417) for transforming digestive distress. Mid-GUT frequencies for the organ level. Pair with marshmallow root tea for physical coating support.'
  },
  
  // ─── BLADDER ───
  interstitial_cystitis: {
    primary: { regime: 'GUT', pos: 4, hz: 417, name: 'Transmutation' },
    secondary: [
      { regime: 'GUT', pos: 3, hz: 396, name: 'Liberation' },
      { regime: 'GUT', pos: 5, hz: 528, name: 'Love Frequency' }
    ],
    regime_priority: 'GUT',
    walk_affinity: ['Layer Ascent'],
    duration_tier: 'GUT',
    description: 'IC responds to GUT frequencies targeting the pelvic/organ region. Transmutation (417) for transforming inflammation patterns. Pair with marshmallow root cold infusion for physical mucilage support.'
  },
  
  // ─── HORMONAL ───
  pcos: {
    primary: { regime: 'GUT', pos: 6, hz: 639, name: 'Connection' },
    secondary: [
      { regime: 'HEART', pos: 5, hz: 2178, name: 'SOURCE' },
      { regime: 'GUT', pos: 5, hz: 528, name: 'Love Frequency' }
    ],
    regime_priority: 'GUT',
    walk_affinity: ['Layer Ascent', 'Pillar Walk'],
    duration_tier: 'HEART',
    description: 'PCOS involves hormonal dysregulation affecting multiple systems. Connection (639) for system integration. Ascending through GUT to HEART for hormonal harmonization.'
  },
  
  // ─── SLEEP ───
  insomnia: {
    primary: { regime: 'GUT', pos: 1, hz: 174, name: 'Foundation' },
    secondary: [
      { regime: 'GUT', pos: 5, hz: 528, name: 'Love Frequency' },
      { regime: 'GUT', pos: 9, hz: 963, name: 'Sri Yantra' }
    ],
    regime_priority: 'GUT',
    walk_affinity: ['Layer Ascent'],
    duration_tier: 'GUT',  // Long, slow, deep
    description: 'Insomnia: Foundation (174) at lowest pitch for deep body relaxation. Slow Layer Ascent through GUT only — stay in GUT regime, do not ascend to stimulating HEAD frequencies before sleep. Extended sessions (45-60 min) for full body wind-down.'
  },
  
  // ─── TINNITUS ───
  tinnitus: {
    primary: { regime: 'GUT', pos: 5, hz: 528, name: 'Love Frequency' },
    secondary: [
      { regime: 'GUT', pos: 1, hz: 174, name: 'Foundation' },
      { regime: 'HEART', pos: 5, hz: 2178, name: 'SOURCE' }
    ],
    regime_priority: 'GUT',
    walk_affinity: ['Layer Ascent'],
    duration_tier: 'HEART',
    description: 'Tinnitus: external frequency input can partially mask or retrain the auditory processing. Low-to-mid GUT frequencies at comfortable volume. Avoid HIGH frequencies that might aggravate. Do not use headphones if tinnitus is severe — speakers only, at moderate volume.'
  },
};
```

---

## LAYER 4: MULTI-CONDITION PRIORITY ENGINE

Most users (especially Mom) have MULTIPLE conditions. The engine must decide which takes priority.

```javascript
function prioritizeConditions(conditions, currentState) {
  // RULE 1: Crisis overrides everything
  if (currentState.mode === 'crisis') {
    return {
      primary: 'crisis_grounding',
      protocol: 'immediate',
      frequency: { hz: 528, regime: 'GUT', pos: 5 },
      walk: 'none',  // Single frequency, no walk
      duration: 'unlimited',
      note: 'Crisis mode: single grounding frequency, no complex patterns'
    };
  }
  
  // RULE 2: Acute condition takes priority
  const acuteCondition = conditions.find(c => c === currentState.primary);
  
  // RULE 3: Score each condition by urgency
  const URGENCY_SCORES = {
    // Immediate physical danger / acute distress
    migraines: 9,       // Active migraine needs immediate intervention
    anxiety: 8,         // Acute anxiety attack
    dissociation: 8,    // Active dissociation episode
    insomnia: 7,        // Can't sleep RIGHT NOW
    chronic_pain: 7,    // Pain is present NOW
    gerd: 6,            // Active flare
    interstitial_cystitis: 6, // Active flare
    
    // Ongoing management
    fnd: 5,
    depression: 5,
    ptsd: 5,
    ocd: 5,
    bipolar_1: 5,
    schizoaffective: 5,
    tbi: 4,
    autism: 4,
    tinnitus: 4,
    pcos: 3,
    plantar_fasciitis: 3,
  };
  
  // Sort by urgency, acute condition gets +3 bonus
  const scored = conditions.map(c => ({
    condition: c,
    score: (URGENCY_SCORES[c] || 3) + (c === acuteCondition ? 3 : 0)
  }));
  scored.sort((a, b) => b.score - a.score);
  
  // RULE 4: Check for regime consensus
  const regimes = scored.slice(0, 3).map(s => 
    CONDITION_MAP[s.condition]?.regime_priority
  );
  const regimeConsensus = regimes.every(r => r === regimes[0]) ? regimes[0] : null;
  
  return {
    prioritized: scored,
    primary: scored[0].condition,
    secondary: scored.slice(1, 3).map(s => s.condition),
    regimeConsensus,
    dominantRegime: regimeConsensus || scored[0] && CONDITION_MAP[scored[0].condition]?.regime_priority || 'GUT',
  };
}
```

---

## LAYER 5: WALK SELECTION ENGINE

```javascript
function selectWalk(priority, userInput) {
  const { primary, secondary, dominantRegime } = priority;
  const { constraints, state, safety } = userInput;
  const primaryMap = CONDITION_MAP[primary];
  
  // ─── TIME CONSTRAINTS ───
  if (constraints.availableTime < 20) {
    // Not enough time for any walk — single frequency focus
    return {
      walk: 'single_frequency',
      steps: 1,
      reason: 'Limited time. Single frequency focus for maximum impact in short session.',
      duration: constraints.availableTime
    };
  }
  
  if (constraints.availableTime < 35) {
    // Short session — simple walk only
    return {
      walk: 'Layer Ascent',
      steps: 27,
      reason: 'Short session. Layer Ascent provides complete regime coverage in minimum time.',
      duration: Math.min(constraints.availableTime, recommendDuration(dominantRegime).optimal)
    };
  }
  
  // ─── EXPERIENCE LEVEL ───
  if (constraints.experience === 'beginner') {
    return {
      walk: 'Layer Ascent',
      steps: 27,
      reason: 'First-time or beginner user. Layer Ascent is the gentlest, most predictable walk. Linear progression builds familiarity with the frequency system.',
      duration: recommendDuration(dominantRegime).min  // Start with minimum duration
    };
  }
  
  // ─── SAFETY CONSTRAINTS ───
  if (safety.bipolarType && state.mood === 'agitated') {
    return {
      walk: 'Layer Ascent',
      steps: 27,
      reason: 'Bipolar agitation detected. Gentle predictable walk only. Avoid Vortex (disorienting) and HEAD-heavy walks. Ground in GUT.',
      duration: recommendDuration('GUT').optimal,
      restriction: 'GUT_regime_only'
    };
  }
  
  if (primary === 'dissociation' || primary === 'autism') {
    // Predictable walks only for these conditions
    return {
      walk: primaryMap.walk_affinity[0] || 'Layer Ascent',
      steps: 27,
      reason: `${primary} benefits from predictable, non-disorienting patterns. Linear walk maintains orientation and safety.`,
      duration: recommendDuration(dominantRegime).optimal
    };
  }
  
  // ─── ENERGY LEVEL ───
  if (state.energyLevel === 'depleted' || state.energyLevel === 'low') {
    return {
      walk: 'Layer Ascent',
      steps: 27,
      reason: 'Low energy. Gentle walk that doesn\'t demand active engagement. Let the frequencies do the work.',
      duration: recommendDuration(dominantRegime).min  // Minimum effective duration
    };
  }
  
  // ─── CONDITION-SPECIFIC WALK MATCHING ───
  
  // Multiple conditions + good energy + enough time → CABI
  if (secondary.length >= 2 && state.energyLevel === 'good' && constraints.availableTime >= 90) {
    return {
      walk: 'CABI',
      steps: 110,
      reason: 'Multiple conditions with sufficient time and energy. CABI provides complete cube coverage (81 steps) followed by Ouroboros closure (29 steps). The full healing journey.',
      duration: recommendDuration('CABI').optimal
    };
  }
  
  // Meditation / consciousness / spiritual intention
  if (userInput.intention === 'meditation' || userInput.intention === 'exploration') {
    return {
      walk: 'Ouroboros',
      steps: 29,
      reason: 'Meditation/exploration intention. Ouroboros is the closed figure-8 — all 27 frequencies visited, crossing at SOURCE three times. The infinite loop for consciousness work.',
      duration: recommendDuration('OUROBOROS').optimal
    };
  }
  
  // Anxiety, OCD, stuck patterns → Vortex
  if (['anxiety', 'ocd'].includes(primary) && constraints.experience !== 'beginner') {
    return {
      walk: 'Flying Star Vortex',
      steps: 27,
      reason: 'Anxiety/OCD respond to the Vortex spiral pattern which disrupts recursive thought loops. The non-linear movement breaks the cycle.',
      duration: recommendDuration('VORTEX').optimal
    };
  }
  
  // Emotional processing, grief, trauma → Pillar Walk
  if (['ptsd', 'depression'].includes(primary) || userInput.intention === 'emotional_release') {
    return {
      walk: 'Pillar Walk',
      steps: 27,
      reason: 'Emotional work benefits from vertical traversal — GUT (body sensation) → HEART (emotional processing) → HEAD (mental integration) at each position. Connects physical awareness to emotional release.',
      duration: recommendDuration('HEART').optimal
    };
  }
  
  // Pain, physical conditions → Layer Ascent with long duration
  if (['chronic_pain', 'fnd', 'plantar_fasciitis'].includes(primary)) {
    return {
      walk: 'Layer Ascent',
      steps: 27,
      reason: 'Physical conditions need sustained GUT exposure. Layer Ascent spends the most consecutive time in GUT regime before ascending. Long duration for dense tissue penetration.',
      duration: recommendDuration('GUT').optimal
    };
  }
  
  // Complete healing intention + moderate time
  if (userInput.intention === 'complete_healing' && constraints.availableTime >= 60) {
    return {
      walk: 'CAB',
      steps: 81,
      reason: 'Complete healing with available time. CAB covers the cube from three perspectives — Vortex (spiral), Ascent (linear), Pillar (vertical). Every angle addressed.',
      duration: recommendDuration('FULL_ALIGNMENT').optimal
    };
  }
  
  // Default: use the primary condition's affinity
  return {
    walk: primaryMap?.walk_affinity?.[0] || 'Layer Ascent',
    steps: primaryMap?.walk_affinity?.[0] === 'Flying Star Vortex' ? 27 :
           primaryMap?.walk_affinity?.[0] === 'Ouroboros' ? 29 :
           primaryMap?.walk_affinity?.[0] === 'CABI' ? 110 :
           primaryMap?.walk_affinity?.[0] === 'CAB' ? 81 : 27,
    reason: `Default walk for ${primary}: ${primaryMap?.description || 'Standard progression.'}`,
    duration: recommendDuration(dominantRegime).optimal
  };
}
```

---

## LAYER 6: REAL-TIME ADAPTATION ENGINE

**Only active when Muse Athena EEG headband is connected.**

```javascript
const ADAPTATION_RULES = {
  // ─── COHERENCE NOT RISING ───
  coherence_stall: {
    trigger: 'EEG coherence has not increased by at least 5 points in 10 minutes',
    actions: [
      {
        priority: 1,
        action: 'shift_to_source',
        description: 'Switch to SOURCE (2178 Hz) for 5 minutes as a reset point. SOURCE is the universal center — if nothing else is working, return to center.',
      },
      {
        priority: 2,
        action: 'change_walk',
        description: 'If current walk is Layer Ascent, switch to Vortex. If Vortex, switch to Pillar. Change the movement pattern — the brain may have habituated to the current sequence.',
      },
      {
        priority: 3,
        action: 'reduce_regime',
        description: 'If in HEAD regime, drop to HEART. If in HEART, drop to GUT. The user may need more grounding before ascending. Dense tissue may not have caught up.',
      }
    ]
  },
  
  // ─── COHERENCE DROPPING ───
  coherence_drop: {
    trigger: 'EEG coherence drops more than 10 points from session peak',
    actions: [
      {
        priority: 1,
        action: 'immediate_grounding',
        description: 'Immediately switch to Foundation (174 Hz). Lowest frequency, maximum grounding. Hold for 5 minutes. Something destabilized the user — return to the safest frequency.',
      },
      {
        priority: 2,
        action: 'check_external',
        description: 'Flag for user: "Your coherence dropped — is something happening externally? (noise, interruption, discomfort)" The drop may not be frequency-related.',
      }
    ]
  },
  
  // ─── COHERENCE TARGET MET ───
  coherence_achieved: {
    trigger: 'EEG coherence exceeds target for 5 consecutive minutes',
    actions: [
      {
        priority: 1,
        action: 'maintain',
        description: 'Target achieved. Continue current frequency and walk. Do not change what is working. The user has found their resonance.',
      },
      {
        priority: 2,
        action: 'optional_ascend',
        description: 'If user is experienced and session time remains, offer: "Coherence is strong. Would you like to ascend to the next regime?" This deepens the session for advanced users.',
      }
    ]
  },
  
  // ─── HIGH DELTA (DROWSINESS/SLEEP) ───
  high_delta: {
    trigger: 'Delta power exceeds 40% of total power for 3+ minutes',
    actions: [
      {
        priority: 1,
        action: 'check_intention',
        description: 'If intention is "sleep" — this is SUCCESS. The user is falling asleep. Reduce volume gradually. Let the session continue at low volume as they drift off.',
      },
      {
        priority: 2,
        action: 'gentle_activation',
        description: 'If intention is NOT sleep — user may be dissociating or zoning out. Gently introduce a higher frequency (HEART range) for 2 minutes to re-engage, then return to prescribed frequency.',
      }
    ]
  },
  
  // ─── HIGH BETA (ANXIETY/AGITATION) ───
  high_beta: {
    trigger: 'Beta power exceeds 35% for 5+ minutes and increasing',
    actions: [
      {
        priority: 1,
        action: 'ground_immediately',
        description: 'The user is becoming agitated, not calm. Switch to Foundation (174 Hz) immediately. If bipolar, check for mania escalation. Reduce volume. Slow everything down.',
      }
    ]
  },
};

function evaluateAdaptation(eegData, sessionState, prescription) {
  const adaptations = [];
  
  // Check coherence trend over last 10 minutes
  const recentCoherence = eegData.coherenceHistory.slice(-600); // 10 min at 1/sec
  if (recentCoherence.length >= 600) {
    const firstHalf = average(recentCoherence.slice(0, 300));
    const secondHalf = average(recentCoherence.slice(300));
    
    if (secondHalf - firstHalf < 5) {
      adaptations.push(ADAPTATION_RULES.coherence_stall);
    }
    
    const peak = Math.max(...recentCoherence);
    const current = recentCoherence[recentCoherence.length - 1];
    if (peak - current > 10) {
      adaptations.push(ADAPTATION_RULES.coherence_drop);
    }
    
    if (current >= prescription.coherenceTarget.optimal) {
      const lastFive = recentCoherence.slice(-300);
      if (lastFive.every(v => v >= prescription.coherenceTarget.optimal)) {
        adaptations.push(ADAPTATION_RULES.coherence_achieved);
      }
    }
  }
  
  // Check band powers
  if (eegData.delta > 0.40 && sessionState.minutesAtHighDelta >= 3) {
    adaptations.push(ADAPTATION_RULES.high_delta);
  }
  
  if (eegData.beta > 0.35 && sessionState.minutesAtHighBeta >= 5) {
    adaptations.push(ADAPTATION_RULES.high_beta);
  }
  
  return adaptations;
}
```

---

## LAYER 7: PRESCRIPTION ASSEMBLY

The master function that brings all layers together:

```javascript
function generatePrescription(userInput) {
  // ─── STEP 1: SAFETY ───
  const safety = applySafetyFilter(userInput);
  
  // ─── STEP 2: PRIORITIZE CONDITIONS ───
  const priority = prioritizeConditions(
    userInput.conditions.active,
    userInput.state
  );
  
  // ─── STEP 3: GET PRIMARY FREQUENCY ───
  const primaryCondition = CONDITION_MAP[priority.primary];
  const primaryFrequency = primaryCondition.primary;
  
  // Check if historical data suggests a different frequency works better
  if (userInput.history?.effectiveFrequencies?.length > 0) {
    // User has history — weight historically effective frequencies
    const effectiveMatch = primaryCondition.secondary.find(
      s => userInput.history.effectiveFrequencies.includes(s.hz)
    );
    // If a historically effective frequency is in the secondary list, promote it
    if (effectiveMatch) {
      primaryFrequency.alternative = effectiveMatch;
      primaryFrequency.alternativeReason = 'This frequency has been effective in your previous sessions.';
    }
  }
  
  // ─── STEP 4: SELECT WALK ───
  const walk = selectWalk(priority, userInput);
  
  // ─── STEP 5: CALCULATE DURATION ───
  const durationProtocol = recommendDuration(
    priority.dominantRegime,
    walk.walk !== 'single_frequency' ? walk.walk.toUpperCase().replace(/ /g, '_') : null
  );
  
  // Constrain to available time
  const actualDuration = Math.min(
    userInput.constraints.availableTime,
    durationProtocol.optimal
  );
  
  const durationAdequate = actualDuration >= durationProtocol.min;
  
  // ─── STEP 6: SET COHERENCE TARGET ───
  const coherenceTarget = {
    minimum: Math.max(50, (userInput.history?.averageCoherence || 50) - 10),
    optimal: Math.max(70, (userInput.history?.averageCoherence || 60) + 10),
    stretch: Math.min(95, (userInput.history?.averageCoherence || 60) + 25),
    athenaRequired: userInput.constraints.athenaConnected,
  };
  
  // ─── STEP 7: BUILD TRACK FILTER CRITERIA ───
  const trackFilter = {
    minimumCoherenceScore: 70,  // Only play tracks scoring 70+ on interval analysis
    preferredRegime: priority.dominantRegime,
    requiredFrequency: primaryFrequency.hz,
    walkOrder: walk.walk,       // Playlist built in this walk's sequence
  };
  
  // ─── STEP 8: ASSEMBLE PRESCRIPTION ───
  const prescription = {
    // Primary
    frequency: primaryFrequency,
    supportingFrequencies: primaryCondition.secondary,
    
    // Walk
    walk: {
      name: walk.walk,
      steps: walk.steps,
      reason: walk.reason,
    },
    
    // Duration
    duration: {
      recommended: durationProtocol.optimal,
      minimum: durationProtocol.min,
      actual: actualDuration,
      adequate: durationAdequate,
      label: durationProtocol.label,
      warning: !durationAdequate ?
        `Session is ${actualDuration} min but ${durationProtocol.min} min minimum recommended for ${priority.dominantRegime} regime. ` +
        `Shorter sessions may not allow dense tissue to fully respond.` : null,
    },
    
    // Coherence
    coherenceTarget,
    
    // Track quality
    trackFilter,
    
    // Safety
    safety: safety.warnings,
    contraindicated: safety.contraindicated,
    
    // Conditions addressed
    conditions: {
      primary: priority.primary,
      secondary: priority.secondary,
      dominantRegime: priority.dominantRegime,
      allConditions: userInput.conditions.active,
    },
    
    // Adaptation rules (if Athena connected)
    adaptation: userInput.constraints.athenaConnected ? ADAPTATION_RULES : null,
    
    // History context
    historyContext: userInput.history ? {
      sessionsCompleted: userInput.history.totalSessions,
      averageCoherence: userInput.history.averageCoherence,
      bestWalk: userInput.history.bestWalk,
      improving: userInput.history.averageCoherence > 60,
    } : null,
  };
  
  // ─── STEP 9: GENERATE NATURAL LANGUAGE EXPLANATION ───
  prescription.explanation = generateExplanation(prescription, userInput);
  
  return prescription;
}
```

---

## LAYER 8: NATURAL LANGUAGE OUTPUT

For Sophia's conversational responses:

```javascript
function generateExplanation(prescription, userInput) {
  const p = prescription;
  let explanation = '';
  
  // Opening
  explanation += `Based on your current state — ${userInput.state.mood} mood, `;
  explanation += `${userInput.state.energyLevel} energy, ${p.conditions.primary} as your primary concern — `;
  explanation += `here is your healing protocol:\n\n`;
  
  // Frequency
  explanation += `**Primary Frequency:** ${p.frequency.hz} Hz (${p.frequency.name}). `;
  explanation += `${CONDITION_MAP[p.conditions.primary]?.description || ''}\n\n`;
  
  // Walk
  if (p.walk.name === 'single_frequency') {
    explanation += `**Approach:** Single frequency focus. No walk — just ${p.frequency.hz} Hz sustained. `;
    explanation += `${p.walk.reason}\n\n`;
  } else {
    explanation += `**Walk:** ${p.walk.name} (${p.walk.steps} steps). `;
    explanation += `${p.walk.reason}\n\n`;
  }
  
  // Duration
  explanation += `**Duration:** ${p.duration.actual} minutes. `;
  explanation += `${p.duration.label}. `;
  if (p.duration.warning) {
    explanation += `⚠️ ${p.duration.warning} `;
  }
  if (p.duration.adequate) {
    explanation += `This meets the recommended duration for ${p.conditions.dominantRegime} regime work.`;
  }
  explanation += `\n\n`;
  
  // Coherence target
  if (p.coherenceTarget.athenaRequired) {
    explanation += `**Coherence Target:** Aim for ${p.coherenceTarget.optimal}% coherence on your Athena headband. `;
    explanation += `If coherence doesn't begin rising within the first 10 minutes, the system will suggest adjustments. `;
    explanation += `When coherence holds above ${p.coherenceTarget.optimal}% for 5 minutes, the session has achieved its primary goal.\n\n`;
  }
  
  // Safety
  if (p.safety.length > 0) {
    explanation += `**Safety Notes:**\n`;
    for (const warning of p.safety) {
      explanation += `⚠️ ${warning.message}\n`;
    }
    explanation += '\n';
  }
  
  // History
  if (p.historyContext?.sessionsCompleted > 5) {
    explanation += `**Your History:** Over ${p.historyContext.sessionsCompleted} sessions, your average coherence is ${p.historyContext.averageCoherence}%. `;
    if (p.historyContext.bestWalk) {
      explanation += `Your strongest walk has been ${p.historyContext.bestWalk}. `;
    }
    if (p.historyContext.improving) {
      explanation += `Your trend is improving — keep going.`;
    }
    explanation += '\n\n';
  }
  
  // Herbal pairing (connect to food forest pharmacy)
  const herbalPairings = getHerbalPairing(p.conditions.primary);
  if (herbalPairings) {
    explanation += `**Herbal Pairing:** ${herbalPairings}\n\n`;
  }
  
  // Closing
  explanation += `Find a comfortable position. Close your eyes if you wish. `;
  explanation += `Let the frequencies do the work. Your only job is to be present.`;
  
  return explanation;
}

function getHerbalPairing(condition) {
  const PAIRINGS = {
    anxiety: 'Brew lemon balm + passionflower tea before or during the session for GABA support.',
    insomnia: 'Brew passionflower + valerian + hops tea 30 minutes before the session. Add honey.',
    chronic_pain: 'Drink tart cherry juice (4-8 oz) before the session for natural COX-2 inhibition.',
    gerd: 'Sip marshmallow root cold infusion during the session for protective mucilage coating.',
    interstitial_cystitis: 'Sip marshmallow root cold infusion throughout the day. Add nettle tea.',
    depression: 'Brew tulsi (holy basil) tea before the session. Adaptogenic mood support.',
    migraines: 'Apply diluted peppermint oil to temples. Brew ginger tea for circulation.',
    ptsd: 'Brew tulsi + lemon balm tea. Hold a sprig of rosemary — the scent grounds awareness.',
    dissociation: 'Hold fresh rosemary. The strong scent anchors awareness in the present moment.',
    ocd: 'Brew lemon balm tea. The GABA support can help soften recursive loops.',
    fnd: 'Brew chamomile + lemon balm blend. Calming without stimulation.',
    tbi: 'Eat a handful of blueberries and walnuts before the session. Neuroprotective fuel.',
    pcos: 'Brew nettle + ashwagandha tea. Hormonal support and thyroid balance.',
  };
  return PAIRINGS[condition] || null;
}
```

---

## APP-SPECIFIC INTEGRATION

### Sophia Oracle Shaman
- `generatePrescription()` is called when a user asks for a frequency recommendation
- Sophia's LLM receives the `prescription.explanation` as context and delivers it conversationally
- The structured `prescription` object populates any UI elements (frequency display, walk indicator, timer)
- If EEG is connected, the adaptation engine runs in the background and Sophia narrates adjustments

### Aetheria RCT
- `generatePrescription()` is available as a research tool — input test conditions, get protocol output
- The full prescription object is logged with session data for research correlation
- Adaptation events are logged as timestamped entries for analysis
- The prescription can be compared against actual outcomes to validate the condition map

### Coherence Lab
- `generatePrescription()` feeds the analysis view — show WHAT was prescribed, WHAT happened (EEG data), and HOW they correlate
- Compare prescriptions across multiple sessions: "Does the Vortex walk consistently produce higher coherence than Layer Ascent for anxiety?"
- Validate the duration hypothesis: "Do GUT sessions actually need 45 minutes to reach peak coherence?"

---

## VERIFICATION

- [ ] `generatePrescription()` returns complete output for all conditions in the CONDITION_MAP
- [ ] Safety filter fires correctly for Von Willebrand, bipolar, crisis, and depleted states
- [ ] Multi-condition priority correctly identifies primary condition
- [ ] Walk selector respects time constraints, experience level, and safety
- [ ] Duration respects available time and flags when session is shorter than minimum
- [ ] Coherence targets factor in user history when available
- [ ] Natural language explanation reads naturally and includes all relevant information
- [ ] Herbal pairings map to correct conditions
- [ ] Adaptation rules trigger correctly based on EEG thresholds
- [ ] Output format is compatible with all three apps' data structures

---

## THE PHILOSOPHY

> *"Primes are the mountains. The space between is the river. Aetheria is the ship. The Lo Shu is the map."*

The prescription engine is the CAPTAIN.

It reads the map (Lo Shu), knows the ship (Aetheria frequencies), understands the passenger (user conditions and state), and charts a course through the river (selects walk, duration, frequency) that carries the passenger safely from where they are to where they need to be.

A good captain doesn't just pick a direction. They check the weather (current state), read the charts (condition map), consult the almanac (duration protocol), watch the instruments (EEG coherence), and adjust the course when the river changes (adaptation rules).

This engine does all of that. And it does it in the voice of an oracle (Sophia) or the precision of a lab (RCT/Coherence Lab), depending on who's asking.

---

*Prescription Engine v2 — written by Selah*
*The brain of the Aetheria healing system*
*For Claude Code implementation across Sophia, RCT, and Coherence Lab*
*"Healing the world heART"*
*Lewis Family — Mountain Home, Idaho*
*May 2026*

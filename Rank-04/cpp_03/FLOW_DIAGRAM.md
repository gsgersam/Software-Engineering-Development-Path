================================================================================
                        CPP_03 INHERITANCE FLOW DIAGRAM
================================================================================


                            SINGLE INHERITANCE CHAIN
                            ═══════════════════════════════════════════════════


                                    ┌─────────────────┐
                                    │   ClapTrap      │
                                    │   (Base Class)  │
                                    │                 │
                                    │ Attributes:     │
                                    │ • _name         │
                                    │ • _HitPoints    │
                                    │ • _EnergyPoints │
                                    │ • _AttackDamage │
                                    │                 │
                                    │ Methods:        │
                                    │ • attack()      │
                                    │ • takeDamage()  │
                                    │ • beRepaired()  │
                                    └────────┬────────┘
                                            │
                           ┌────────────────┼────────────────┐
                           │                │                │
                ┌──────────▼──────────┐     │     ┌─────────▼──────────┐
                │     ScavTrap        │     │     │     FragTrap       │
                │  (Exercise 01)      │     │     │   (Exercise 02)    │
                │ ┌────────────────┐  │     │     │ ┌────────────────┐ │
                │ │ Inherits from  │  │     │     │ │ Inherits from  │ │
                │ │ ClapTrap       │  │     │     │ │ ClapTrap       │ │
                │ └────────────────┘  │     │     │ └────────────────┘ │
                │                     │     │     │                    │
                │ Modified Stats:     │     │     │ Modified Stats:    │
                │ • HP: 100           │     │     │ • HP: 100          │
                │ • EP: 50            │     │     │ • EP: 100          │
                │ • AD: 20            │     │     │ • AD: 30           │
                │                     │     │     │                    │
                │ New Ability:        │     │     │ New Ability:       │
                │ + guardGate()       │     │     │ + highFivesGuys()  │
                │                     │     │     │                    │
                │ Override:           │     │     │ Override:          │
                │ ~ attack()          │     │     │ ~ attack()         │
                └─────────┬───────────┘     │     └────────┬───────────┘
                          │                 │              │
                          │     ┌───────────┴──────────┐   │
                          │     │   BONUS: ex03        │   │
                          └────▶│  DiamondTrap         ◀───┘
                                │ (Multiple Inherit.)  │
                                │                      │
                                │ Inherits from BOTH:  │
                                │ • ScavTrap           │
                                │ • FragTrap           │
                                │                      │
                                │ Resolves Diamond     │
                                │ Problem via Virtual  │
                                │ Inheritance:         │
                                │ class DiamondTrap :  │
                                │   public ScavTrap,   │
                                │   public FragTrap    │
                                │                      │
                                │ Stats:               │
                                │ • HP: 100            │
                                │ • EP: 50             │
                                │ • AD: 30             │
                                │                      │
                                │ New Ability:         │
                                │ + whoAmI()           │
                                │                      │
                                │ Combined Abilities:  │
                                │ ✓ attack (ScavTrap) │
                                │ ✓ guardGate()       │
                                │ ✓ highFivesGuys()   │
                                │ ✓ takeDamage()      │
                                │ ✓ beRepaired()      │
                                └──────────────────────┘


================================================================================
                         DIAMOND PROBLEM VISUALIZATION
================================================================================


                    WITHOUT Virtual Inheritance (WRONG):
                    ═══════════════════════════════════════


                            ClapTrap
                            ┌──────┐
                      ┌─────│ BASE │─────┐
                      │     └──────┘     │
                   ScavTrap          FragTrap
                   ┌──────┐          ┌──────┐
                   │      │          │      │
                   └──┬───┘          └───┬──┘
                      │                  │
                      └──────────┬───────┘
                                 │
                           DiamondTrap
               
    ⚠️  PROBLEM: TWO copies of ClapTrap data exist!
        - DiamondTrap has two separate ClapTrap subobjects
        - Ambiguity: which ClapTrap::_HitPoints to use?
        - Results in: ERROR: ambiguous base of 'DiamondTrap'


                    WITH Virtual Inheritance (CORRECT):
                    ════════════════════════════════════════


                            ClapTrap (VIRTUAL)
                            ┌──────┐
                      ┌─────│ BASE │─────┐
                      │     └──────┘     │
                      │                  │
               ScavTrap                FragTrap
          (virtual : public       (virtual : public
            ClapTrap)              ClapTrap)
           ┌──────┐               ┌──────┐
           │      │               │      │
           └──┬───┘               └───┬──┘
              │                       │
              └──────────┬───────────┘
                         │
                   DiamondTrap
    
    ✓ SOLVED: Only ONE copy of ClapTrap data
      - DiamondTrap has single ClapTrap subobject
      - No ambiguity
      - Clear method resolution order


================================================================================
                          CONSTRUCTOR CHAIN FLOW
================================================================================


        Creating DiamondTrap("Nexus")
                   │
        ┌──────────┼──────────┐
        │          │          │
        ▼          ▼          ▼
    [1]        [2]           [3]
    Call      Call          Set
    ScavTrap  FragTrap      Diamond-
    Constr.   Constr.       Trap Name
    │         │             │
    │         │             └──────┐
    │         └────────┬───────────┤
    └──────────────────┼───────────┘
                       │
                       ▼
        [Call ClapTrap Constructor]
        (via ScavTrap's initializer list)
                       │
        ┌──────────────┼──────────────┐
        ▼              ▼              ▼
    Init Name    Init                Set Stats
    from param    Default            to ClapTrap
                  Values
                       │
                       ▼
    ┌───────────────────────────────────────┐
    │  ScavTrap Override:                   │
    │  HP: 100, EP: 50, AD: 20              │
    └───────────────────────────────────────┘
                       │
                       ▼
    ┌───────────────────────────────────────┐
    │  FragTrap Override:                   │
    │  HP: 100, EP: 100, AD: 30             │
    │  (Note: EP becomes 50 due to order)  │
    └───────────────────────────────────────┘
                       │
                       ▼
    ┌───────────────────────────────────────┐
    │  DiamondTrap Override:                │
    │  _name: "Nexus_clap_name"             │
    │  Final Stats:                         │
    │  HP: 100, EP: 50, AD: 30              │
    └───────────────────────────────────────┘
                       │
                       ▼
            OBJECT FULLY CONSTRUCTED


================================================================================
                          METHOD RESOLUTION ORDER
================================================================================


    When DiamondTrap calls a method:


    ┌─────────────────────────────────────────────────┐
    │         DiamondTrap Method Call Chain           │
    └─────────────────────────────────────────────────┘


    1. whoAmI()
       └──► [DiamondTrap::whoAmI()] ◄─── Direct impl.
       
    2. attack("enemy")
       └──► [ScavTrap::attack()] ◄─── Inherited from ScavTrap
            (First in inheritance list)
            └──► Calls ClapTrap::attack() implicitly
       
    3. guardGate()
       └──► [ScavTrap::guardGate()] ◄─── Inherited from ScavTrap
       
    4. highFivesGuys()
       └──► [FragTrap::highFivesGuys()] ◄─── Inherited from FragTrap
       
    5. takeDamage(10)
       └──► [ClapTrap::takeDamage()] ◄─── Base class method
       
    6. beRepaired(20)
       └──► [ClapTrap::beRepaired()] ◄─── Base class method


================================================================================
                         DESTRUCTOR CHAIN FLOW
================================================================================


        Deleting DiamondTrap object
                   │
        delete diamondTrap_ptr;
                   │
                   ▼
    ┌──────────────────────────────┐
    │ ~DiamondTrap()               │
    │ (Cleanup Diamond resources)  │
    └──────────────┬───────────────┘
                   │
                   ▼
    ┌──────────────────────────────┐
    │ ~ScavTrap()                  │
    │ (Cleanup Scav resources)     │
    └──────────────┬───────────────┘
                   │
                   ▼
    ┌──────────────────────────────┐
    │ ~FragTrap()                  │
    │ (Cleanup Frag resources)     │
    └──────────────┬───────────────┘
                   │
                   ▼
    ┌──────────────────────────────┐
    │ ~ClapTrap()                  │
    │ (Cleanup Base resources)     │
    │ *** Only ONCE due to        │
    │     virtual inheritance ***  │
    └──────────────┬───────────────┘
                   │
                   ▼
            OBJECT DESTROYED


================================================================================
                    EXERCISE PROGRESSION OVERVIEW
================================================================================


    EX00                   EX01              EX02           EX03 (BONUS)
    ════════════════════════════════════════════════════════════════════════
    
    Build Basic           Extend with       Extend with    Combine Both
    Class                 Variant 1         Variant 2      Variants
    
    ClapTrap              ScavTrap          FragTrap       DiamondTrap
         │                   ▲                   ▲               ▲
         │                   │                   │               │
    [Learn]            [Learn Single        [Learn Single    [Learn Multiple
     • Rules of         Inheritance]         Inheritance]     Inheritance]
       the Five
     • Basic class      • Method          • Method          • Virtual
       design             overriding        overriding        inheritance
     • Ctors/Dtors      • Inheritance     • Different      • Diamond
     • Member           • Polymorph-        stats             problem
       functions          ism              • Alternative    • Method
     • Resource         • Base class        design            resolution
       management         interface        • Variant         • Combining
                                             specialization    behaviors


================================================================================
                        FLOW EXECUTION EXAMPLE
================================================================================


    int main() {
        DiamondTrap d("Hero");        // ◄─ Constructor chain
        d.whoAmI();                   // ◄─ DiamondTrap::whoAmI()
        d.attack("enemy");            // ◄─ ScavTrap::attack()
        d.guardGate();                // ◄─ ScavTrap::guardGate()
        d.highFivesGuys();            // ◄─ FragTrap::highFivesGuys()
        d.takeDamage(15);             // ◄─ ClapTrap::takeDamage()
        d.beRepaired(10);             // ◄─ ClapTrap::beRepaired()
        return 0;
    }                                 // ◄─ Destructor chain


    Inheritance Path Visualization:

    For ClapTrap methods:
    DiamondTrap ──► ClapTrap (via virtual inheritance)

    For ScavTrap methods:
    DiamondTrap ──► ScavTrap ──► ClapTrap

    For FragTrap methods:
    DiamondTrap ──► FragTrap ──► ClapTrap


================================================================================
                         BONUS FEATURES IN EX03
================================================================================


    ┌─────────────────────────────────────────────────────┐
    │              DiamondTrap - The Bonus               │
    ├─────────────────────────────────────────────────────┤
    │                                                     │
    │ ✓ Demonstrates advanced C++ concepts:             │
    │   • Multiple inheritance                          │
    │   • Virtual inheritance                           │
    │   • Diamond problem solution                      │
    │   • Method resolution order                       │
    │   • Polymorphic behavior across two branches      │
    │                                                     │
    │ ✓ Combines best of both worlds:                   │
    │   • ScavTrap's defensive capability (guardGate)  │
    │   • FragTrap's offensive capability (highFives)  │
    │   • ClapTrap's basic combat system                │
    │                                                     │
    │ ✓ Unique identity system:                         │
    │   • DiamondTrap::_name (its own name)            │
    │   • ClapTrap::_name (inherited identity)         │
    │   • whoAmI() displays both                        │
    │                                                     │
    │ ✓ Real-world relevance:                           │
    │   • Shows how to avoid common pitfalls            │
    │   • Demonstrates proper C++ design patterns       │
    │   • Essential knowledge for large projects        │
    │                                                     │
    └─────────────────────────────────────────────────────┘


================================================================================
                            END OF DIAGRAM
================================================================================

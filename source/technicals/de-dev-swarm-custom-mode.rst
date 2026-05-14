.. _de-dev-swarm-custom-mode:


========================================
How to Add a New Custom Swarm Mode
========================================

This guide explains how to extend the DroneEngage SWARM system with custom formation types and operational modes.


Overview
========

The DroneEngage SWARM system is designed to be extensible. You can add:

- **New Formation Types**: Custom spatial arrangements (e.g., Circle, Grid, Diamond)
- **New Operational Modes**: Broader behavioral patterns beyond formations (e.g., autonomous behaviors, special mission modes)

The system is implemented in three main classes located in ``droneengage_mavlink/src/swarm/``:

- ``CSwarmManager`` - Central manager for swarm state
- ``CSwarmLeader`` - Leader-specific logic for broadcasting to followers
- ``CSwarmFollower`` - Follower-specific logic for receiving leader data and calculating positions


Adding a New Formation Type
============================

Step 1: Define the Formation Constant
-------------------------------------

In ``fcb_swarm_manager.hpp``, add your new formation to the enum and define the constant:

.. code-block:: cpp

    // Add to the enum
    typedef enum
    {
        FORMATION_NO_SWARM      = 0, 
        FORMATION_THREAD        = 1,
        FORMATION_ARROW         = 2, 
        FORMATION_VECTOR        = 3,
        FORMATION_YOUR_CUSTOM   = 4,  // Add your custom formation here
    } ANDRUAV_SWARM_FORMATION;

    // Define the constant (optional, for backward compatibility)
    #define FORMATION_YOUR_CUSTOM   4


Step 2: Implement Leader Broadcast Logic
-----------------------------------------

In ``fcb_swarm_leader.hpp``, declare a new method for your formation:

.. code-block:: cpp

    private:
        void updateFollowersYourCustomFormation();

In ``fcb_swarm_leader.cpp``, implement the method. The leader typically broadcasts its position and attitude to followers:

.. code-block:: cpp

    void CSwarmLeader::updateFollowersYourCustomFormation()
    {
        mavlinksdk::CVehicle &vehicle = mavlinksdk::CVehicle::getInstance();
        
        // Get my attitude and position to share with followers
        const mavlink_attitude_t& attitude = vehicle.getMsgAttitude();
        const mavlink_global_position_int_t& my_gpos = vehicle.getMsgGlobalPositionInt();

        // Loop on follower units
        const std::vector<ANDRUAV_UNIT_FOLLOWER>& follower_units = m_fcb_swarm_manager.getFollowerUnits();
        for (const auto& item : follower_units) 
        {
            mavlink_message_t mavlink_message[2];
            
            const int sys_id = vehicle.getSysId();
            const int comp_id = vehicle.getCompId();

            // Encode position data
            mavlink_msg_global_position_int_encode(sys_id, comp_id, &mavlink_message[0], &my_gpos);
            // Encode attitude data
            mavlink_msg_attitude_encode(sys_id, comp_id, &mavlink_message[1], &attitude);
            
            // Forward info to followers
            de::fcb::CFCBFacade::getInstance().sendSWARM_M(item.party_id, mavlink_message, 2);
        }
    }

Then add your case to the switch statement in ``updateFollowers()``:

.. code-block:: cpp

    void CSwarmLeader::updateFollowers()
    {
        switch (m_fcb_swarm_manager.getFormationAsLeader())
        {
            case FORMATION_THREAD:
                updateFollowersThreadFormation();
                break;
            case FORMATION_ARROW:
                updateFollowerInArrowFormation();
                break;
            case FORMATION_YOUR_CUSTOM:
                updateFollowersYourCustomFormation();  // Add your case
                break;
            default:
                break;
        }
    }


Step 3: Implement Follower Position Calculation
------------------------------------------------

In ``fcb_swarm_follower.hpp``, declare a new method:

.. code-block:: cpp

    private:
        void updateFollowerInYourCustomFormation();

In ``fcb_swarm_follower.cpp``, implement the method. This is where you calculate the follower's target position based on:

- Leader's current position (from ``m_leader_gpos_new``)
- Follower's index (from ``fcb_swarm_manager.getFollowerIndex()``)
- Distance parameters (``m_min_horizontal_distance``, ``m_min_vertical_distance``)
- Your custom formation logic

Example implementation:

.. code-block:: cpp

    void CSwarmFollower::updateFollowerInYourCustomFormation()
    {
        // Get my own location
        mavlinksdk::CVehicle &vehicle = mavlinksdk::CVehicle::getInstance();
        const mavlink_global_position_int_t& my_gpos = vehicle.getMsgGlobalPositionInt();
        
        const double leader_lat = m_leader_gpos_new.lat / 10000000.0f;
        const double leader_lon = m_leader_gpos_new.lon / 10000000.0f;

        // Get current time
        const u_int64_t now = get_time_usec();

        // Test if leader speed is very low, then break
        double speed_sq = m_leader_gpos_new.vx * m_leader_gpos_new.vx + m_leader_gpos_new.vy * m_leader_gpos_new.vy;
        if (speed_sq < 100) 
        {
            return;
        }

        const int follower_index = fcb_swarm_manager.getFollowerIndex();
        
        // YOUR CUSTOM LOGIC HERE
        // Calculate target position based on your formation pattern
        // Example: Calculate bearing and distance for your custom arrangement
        const double base_distance = (follower_index + 1) * m_min_horizontal_distance;
        const double leader_velocity_vector_bearing = getBearingOfVector(m_leader_gpos_new.vx, m_leader_gpos_new.vy);
        
        // Calculate target point using your custom logic
        POINT_2D p = get_point_at_bearing(leader_lat, leader_lon, leader_velocity_vector_bearing, base_distance);

        // Instruct follower to go to target point
        mavlinksdk::CMavlinkCommand::getInstance().gotoGuidedPoint(
            p.latitude, 
            p.longitude, 
            (m_leader_gpos_new.relative_alt + (follower_index + 1) * m_min_vertical_distance * 1000) / 1000.0f
        );

        // Broadcast target location
        CFCBFacade::getInstance().sendFCBTargetLocation("", p.latitude, p.longitude, (double)m_leader_gpos_new.relative_alt, DESTINATION_SWARM_MY_LOCATION);

        // Store latest readings
        m_leader_last_access = now;
        m_leader_gpos_old = m_leader_gpos_new;
    }

Then add your case to the switch statement in ``updateFollower()``:

.. code-block:: cpp

    void CSwarmFollower::updateFollower()
    {
        de::fcb::swarm::CSwarmManager& fcb_swarm_manager = de::fcb::swarm::CSwarmManager::getInstance();

        switch (fcb_swarm_manager.getFormationAsFollower())
        {
            case FORMATION_THREAD:
                updateFollowerInThreadFormation();
                break;
            case FORMATION_ARROW:
                updateFollowerInArrowFormation();
                break;
            case FORMATION_YOUR_CUSTOM:
                updateFollowerInYourCustomFormation();  // Add your case
                break;
            default:
                break;
        }
    }


Step 4: Test Your Formation
---------------------------

1. Build the droneengage_mavlink project
2. Deploy to your test drones
3. Use the WebClient to:
   - Make one drone a leader with your custom formation
   - Add follower drones to the swarm
4. Verify the formation behavior matches your design


Adding a New Operational Mode
==============================

Operational modes are broader than formations and may involve changes to:

- Message handling logic
- State management in ``CSwarmManager``
- Interaction with other systems (mission planner, GCS, etc.)

Step 1: Define Your Mode
--------------------------

Determine what your operational mode does:

- Is it a state that can coexist with formations?
- Does it require new message types?
- What are the entry/exit conditions?

Step 2: Extend CSwarmManager
-----------------------------

Add state variables and methods to ``fcb_swarm_manager.hpp``:

.. code-block:: cpp

    public:
        void enableYourCustomMode(const std::string& parameters);
        void disableYourCustomMode();
        bool isInYourCustomMode() const;

    private:
        bool m_your_custom_mode_enabled = false;
        // Add any additional state variables needed

Implement the logic in ``fcb_swarm_manager.cpp``:

.. code-block:: cpp

    void CSwarmManager::enableYourCustomMode(const std::string& parameters)
    {
        // Parse parameters and initialize your custom mode
        m_your_custom_mode_enabled = true;
        
        // Send notification to GCS
        std::string event = "Custom mode enabled: " + parameters;
        de::fcb::CFCBFacade::getInstance().sendErrorMessage(
            std::string(ANDRUAV_PROTOCOL_SENDER_ALL_GCS), 
            0, 
            ERROR_TYPE_LO7ETTA7AKOM, 
            NOTIFICATION_TYPE_INFO, 
            event
        );
    }

    void CSwarmManager::disableYourCustomMode()
    {
        m_your_custom_mode_enabled = false;
        // Clean up any resources
        
        std::string event = "Custom mode disabled";
        de::fcb::CFCBFacade::getInstance().sendErrorMessage(
            std::string(ANDRUAV_PROTOCOL_SENDER_ALL_GCS), 
            0, 
            ERROR_TYPE_LO7ETTA7AKOM, 
            NOTIFICATION_TYPE_INFO, 
            event
        );
    }

    bool CSwarmManager::isInYourCustomMode() const
    {
        return m_your_custom_mode_enabled;
    }

Step 3: Add Message Handlers
-----------------------------

If your mode requires new message types, add handlers in ``CSwarmManager``:

.. code-block:: cpp

    public:
        void handleYourCustomMessage(const Json_de& andruav_message, const char * full_message, const int & full_message_length);

Implement in ``fcb_swarm_manager.cpp``:

.. code-block:: cpp

    void CSwarmManager::handleYourCustomMessage(const Json_de& andruav_message, const char * full_message, const int & full_message_length)
    {
        const Json_de cmd = andruav_message[ANDRUAV_PROTOCOL_MESSAGE_CMD];
        
        // Validate required fields
        if (!cmd.contains("action") || !cmd["action"].is_string()) return;
        
        const std::string action = cmd["action"].get<std::string>();
        
        if (action == "enable")
        {
            std::string parameters = "";
            if (cmd.contains("params") && cmd["params"].is_string())
            {
                parameters = cmd["params"].get<std::string>();
            }
            enableYourCustomMode(parameters);
        }
        else if (action == "disable")
        {
            disableYourCustomMode();
        }
    }

Register your handler in the message routing logic (typically in the main message handler).

Step 4: Integrate with Existing Logic
-------------------------------------

Modify existing methods to respect your custom mode:

.. code-block:: cpp

    void CSwarmLeader::handleSwarmsAsLeader()
    {
        static u_int64_t previous = 0;
        if (!m_fcb_swarm_manager.isLeader()) return;
        
        // Check if custom mode is active and handle accordingly
        if (m_fcb_swarm_manager.isInYourCustomMode())
        {
            // Custom mode-specific logic
            handleCustomModeAsLeader();
        }
        else
        {
            // Standard formation logic
            const u_int64_t now = get_time_usec();
            if ((now - previous) > DEF_SWARM_LEADER_LOCATION_UPDATE_RATE)
            {
                updateFollowers();
                previous = now;
            }
        }
    }


Step 5: Test Your Operational Mode
-----------------------------------

1. Build and deploy
2. Send test messages to enable/disable your mode
3. Verify behavior through GCS notifications and drone behavior
4. Test edge cases (mode changes, leader transitions, etc.)


Best Practices
==============

- **Use existing patterns**: Follow the structure of Thread and Arrow formations as templates
- **Test incrementally**: Start with a simple formation, then add complexity
- **Handle edge cases**: Consider what happens when:
  - Leader speed is very low
  - Follower loses connection to leader
  - Formation changes dynamically
  - Multiple followers join/leave
- **Add logging**: Use console logging for debugging (see ``_INFO_CONSOLE_TEXT`` macros in existing code)
- **Document parameters**: Clearly document any new parameters your mode requires


Useful Helper Functions
=======================

The following helper functions are available in the codebase:

- ``calcGPSDistance(lat1, lon1, lat2, lon2)`` - Calculate distance between two GPS coordinates
- ``calculateBearing(lat1, lon1, lat2, lon2)`` - Calculate bearing between two points
- ``getBearingOfVector(vx, vy)`` - Calculate bearing from velocity vector
- ``get_point_at_bearing(lat, lon, bearing, distance)`` - Get GPS point at bearing and distance
- ``get_time_usec()`` - Get current time in microseconds

These are defined in ``helpers/gps.hpp`` and ``helpers/helpers.hpp``.


Troubleshooting
===============

**Formation not activating**
  - Verify the formation constant is added to the enum
  - Check the switch statements in both leader and follower
  - Ensure the formation ID matches between enum and constant definitions

**Followers not moving**
  - Check leader is broadcasting data (enable DEBUG logging)
  - Verify follower is receiving MAVLink messages
  - Ensure follower index is set correctly
  - Check speed threshold in your formation logic

**Formation changes not propagating**
  - Verify ``handleMakeSwarm`` calls ``requestFromUnitToChangeFormation`` for existing followers
  - Check follower handles ``SWARM_CHANGE_FORMATION`` action in ``handleFollowHimRequest``

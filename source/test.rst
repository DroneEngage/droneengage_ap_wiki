.. _test

Message: MISSION_UPLOAD
-----------------------

Purpose
  Uploads a mission to the vehicle via server.

Schema
  - `id` (int): Mission ID.
  - `waypoints` (array): List of waypoint objects.

Example
  .. code-block:: json

     {"type":"MISSION_UPLOAD","id":3,"waypoints":[...]}
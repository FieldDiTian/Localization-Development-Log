1. I tested the online adapter. It functions correctly, but the error is very large.
    
2. The primary cause of the large error is an issue with the IMU timestep caused by bursty data delivery.
    
3. The red trajectory is GICP, and the green trajectory is GNSS. The red trajectory occasionally diverges sharply and is then pulled back, which is caused by IMU bursts. The driver needs to be rewritten. Since the current IMU driver already produces bursty output, we may need to rewrite it and then conduct another on-vehicle test. Alternatively, we could stop using the P1 IMU and replace it with another IMU that does not exhibit burst behavior.
    
4. This issue can be mitigated by post-processing the IMU data offline and correcting the timestep alignment. Therefore, GLIO itself does not appear to have a major problem.
    
5. I need an ARTNAS account. I currently do not have the Laguna Seca data, so I cannot build the map.
    
6. We need to align all sensor timestamps to a common time axis. I think using P1 Time as the unified time domain is a reasonable choice. The P1 sensor driver provides mappings from the timestamps of various sensors to P1 Time. However, P1 Time is an artificially constructed common time domain and does not correspond directly to any real-world physical clock.
https://drive.google.com/file/d/1uaUuG0mq6-6zWqgLshWA38Y7htkgGs1v/view?usp=sharing
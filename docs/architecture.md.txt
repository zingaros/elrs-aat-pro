1     # Architecture
2     ## AAT-Pro
3     ### Advanced ExpressLRS Backpack Antenna Tracker
4     
5     Version: 0.2 Draft
6     
7     Status: Architecture Definition
8     
9     Related Documents:
10     
11     - requirements.md
12     - tracking_algorithm.md
13     - backlog.md
14     - hardware_profiles.md
15     - pinout.md
16     - board_profiles.md
17     
18     ---
19     
20     # 1. Executive Summary
21     
22     AAT-Pro is an advanced antenna tracker target built on top of the existing ExpressLRS Backpack ecosystem.
23     
24     The project extends the current AAT implementation while preserving compatibility with existing users and hardware.
25     
26     The primary objectives are:
27     
28     - Reuse existing Backpack infrastructure
29     - Preserve AAT Classic behavior
30     - Support advanced ESP32 hardware
31     - Improve tracking quality
32     - Improve configuration experience
33     - Provide a scalable architecture for future development
34     
35     AAT-Pro is not intended to replace AAT Classic.
36     
37     It is intended to become an advanced member of the Backpack ecosystem.
38     
39     ---
40     
41     # 2. Design Goals
42     
43     ## Primary Goals
44     
45     - Zero regression for existing AAT users
46     - Native Backpack integration
47     - Upstream-friendly architecture
48     - Modular features
49     - Long-range FPV optimization
50     - ArduPilot compatibility
51     
52     ---
53     
54     ## Secondary Goals
55     
56     - Modern WebUI
57     - GPS-aware home positioning
58     - Automatic orientation
59     - Continuous pan tracking
60     - Advanced diagnostics
61     - Future MAVLink support
62     
63     ---
64     
65     # 3. Integration Philosophy
66     
67     AAT-Pro shall be developed as an extension of the existing Backpack architecture.
68     
69     The project shall reuse:
70     
71     - Backpack build system
72     - Backpack WebUI infrastructure
73     - Backpack telemetry routing
74     - Backpack target registration
75     - ExpressLRS hardware target model
76     - Existing firmware update mechanisms
77     
78     The project shall not introduce:
79     
80     - Parallel configuration systems
81     - Independent firmware ecosystem
82     - Custom flashing workflows
83     - Independent target registry
84     
85     Whenever an existing Backpack mechanism exists, it shall be reused.
86     
87     ---
88     
89     # 4. Architecture Audit Findings
90     
91     The ExpressLRS ecosystem already provides infrastructure for:
92     
93     - Hardware targets
94     - Hardware variants
95     - Platform abstraction
96     - Firmware generation
97     - Configuration embedding
98     - Device registration
99     
100     Hardware definitions are managed using:
101     
102     ```text
103     targets.json
104     layout definitions
105     platform variants
106     ```
107     
108     within the existing ExpressLRS target infrastructure. ?1-f18bfb??2-fafdb9??3-ed8fcb?
109     
110     As a result:
111     
112     ```text
113     AAT-Pro shall integrate with
114     the existing target system.
115     ```
116     
117     rather than implementing a new board abstraction framework.
118     
119     ---
120     
121     # 5. Product Variants
122     
123     ## AAT_CLASSIC
124     
125     Purpose:
126     
127     ```text
128     Legacy compatibility
129     ```
130     
131     Target Hardware:
132     
133     ```text
134     ESP8266
135     ESP8285
136     ```
137     
138     Features:
139     
140     ```text
141     Current AAT functionality
142     ```
143     
144     Behavior:
145     
146     ```text
147     Unchanged
148     ```
149     
150     ---
151     
152     ## AAT_PRO_ESP32
153     
154     Purpose:
155     
156     ```text
157     Enhanced tracking
158     ```
159     
160     Target Hardware:
161     
162     ```text
163     ESP32
164     ```
165     
166     Features:
167     
168     ```text
169     GPS
170     Compass
171     WebUI extensions
172     Pan 360
173     Filtering
174     ```
175     
176     ---
177     
178     ## AAT_PRO_ESP32_S3
179     
180     Purpose:
181     
182     ```text
183     Reference platform
184     ```
185     
186     Target Hardware:
187     
188     ```text
189     ESP32-S3
190     ```
191     
192     Features:
193     
194     ```text
195     All advanced features
196     ```
197     
198     including:
199     
200     ```text
201     Prediction
202     Diagnostics
203     Auto latency
204     Simulation
205     MAVLink
206     ```
207     
208     ---
209     
210     # 6. Target Definition Strategy
211     
212     AAT-Pro shall leverage Backpack target definitions.
213     
214     Example future targets:
215     
216     ```text
217     DIY_AAT_CLASSIC
218     
219     DIY_AAT_PRO_ESP32
220     
221     DIY_AAT_PRO_ESP32_S3
222     ```
223     
224     Each target shall define:
225     
226     ```text
227     Platform
228     Capabilities
229     Upload methods
230     Default pin mapping
231     Feature availability
232     ```
233     
234     using existing Backpack mechanisms. ?3-ed8fcb?
235     
236     ---
237     
238     # 7. Capability-Based Architecture
239     
240     The firmware shall rely on capabilities rather than board names.
241     
242     Preferred:
243     
244     ```cpp
245     if(capability.compass)
246     {
247     }
248     ```
249     
250     Not preferred:
251     
252     ```cpp
253     if(board == ESP32S3)
254     {
255     }
256     ```
257     
258     ---
259     
260     ## Core Capabilities
261     
262     ```text
263     CAP_LOCAL_GPS
264     
265     CAP_COMPASS
266     
267     CAP_PAN360
268     
269     CAP_OLED
270     
271     CAP_WEBUI
272     
273     CAP_PID
274     
275     CAP_FILTERING
276     
277     CAP_PREDICTION
278     
279     CAP_AUTO_LATENCY
280     
281     CAP_MAVLINK
282     
283     CAP_DIAGNOSTICS
284     
285     CAP_SIMULATION
286     ```
287     
288     ---
289     
290     # 8. Capability Activation Model
291     
292     Capabilities become active through:
293     
294     ```text
295     Compile-Time Target
296             +
297     Runtime Detection
298             +
299     User Configuration
300     ```
301     
302     Example:
303     
304     ```text
305     ESP32-S3 build
306     
307     +
308     Compass detected
309     
310     +
311     Compass enabled
312     
313     =
314     Compass active
315     ```
316     
317     ---
318     
319     # 9. System Overview
320     
321     ```text
322                         Telemetry Sources
323                                ?
324                  ?????????????????????????????
325                  ?                           ?
326                  ?                           ?
327     
328              CRSF Input                 MAVLink Input
329     
330                  ?
331                  ?
332     
333            TelemetryManager
334     
335                  ?
336                  ?
337     
338              Validation
339     
340                  ?
341                  ?
342     
343           Filtering Engine
344     
345                  ?
346                  ?
347     
348           Prediction Engine
349     
350                  ?
351                  ?
352     
353           Navigation Engine
354     
355                  ?
356                  ?
357     
358                PID Layer
359     
360                  ?
361                  ?
362     
363             Servo Manager
364     
365                  ?
366                  ?
367     
368            Pan + Tilt Output
369     ```
370     
371     ---
372     
373     # 10. Core Modules
374     
375     ## TelemetryManager
376     
377     Responsibilities:
378     
379     ```text
380     CRSF Reception
381     MAVLink Reception
382     Packet Validation
383     Packet Statistics
384     Telemetry Monitoring
385     ```
386     
387     Outputs:
388     
389     ```text
390     TargetState
391     ```
392     
393     ---
394     
395     ## HomeManager
396     
397     Responsibilities:
398     
399     ```text
400     Home Acquisition
401     Home Validation
402     Home Persistence
403     ```
404     
405     Sources:
406     
407     ```text
408     Manual
409     Telemetry
410     Local GPS
411     ```
412     
413     ---
414     
415     ## HeadingManager
416     
417     Responsibilities:
418     
419     ```text
420     Compass Reading
421     Heading Estimation
422     Calibration
423     ```
424     
425     Sources:
426     
427     ```text
428     Manual
429     Compass
430     Compass + GPS
431     ```
432     
433     ---
434     
435     ## NavigationEngine
436     
437     Responsibilities:
438     
439     ```text
440     Distance
441     Bearing
442     Elevation
443     ```
444     
445     Inputs:
446     
447     ```text
448     Home Position
449     Target Position
450     ```
451     
452     Outputs:
453     
454     ```text
455     Distance
456     Bearing
457     Elevation
458     ```
459     
460     ---
461     
462     ## FilteringEngine
463     
464     Responsibilities:
465     
466     ```text
467     GPS Filtering
468     EMA Filtering
469     Servo Deadband
470     Motion Smoothing
471     ```
472     
473     ---
474     
475     ## PredictionEngine
476     
477     Responsibilities:
478     
479     ```text
480     Velocity Estimation
481     Position Projection
482     Latency Compensation
483     Telemetry Loss Extrapolation
484     ```
485     
486     Modes:
487     
488     ```text
489     Off
490     Linear
491     Adaptive
492     ```
493     
494     ---
495     
496     ## PIDController
497     
498     Responsibilities:
499     
500     ```text
501     Pan Stabilization
502     Tilt Stabilization
503     Backlash Compensation
504     ```
505     
506     Two controllers:
507     
508     ```text
509     Pan PID
510     
511     Tilt PID
512     ```
513     
514     ---
515     
516     ## ServoManager
517     
518     Responsibilities:
519     
520     ```text
521     Servo Abstraction
522     Rate Limiting
523     Acceleration Limiting
524     Tilt Easing
525     ```
526     
527     Supported Modes:
528     
529     ```text
530     Standard
531     
532     Continuous 360°
533     ```
534     
535     ---
536     
537     ## SafetyManager
538     
539     Responsibilities:
540     
541     ```text
542     GPS Validation
543     Compass Validation
544     Telemetry Watchdog
545     Zenith Handling
546     Servo Monitoring
547     ```
548     
549     ---
550     
551     ## DisplayManager
552     
553     Responsibilities:
554     
555     ```text
556     OLED Rendering
557     Status Display
558     Field Diagnostics
559     ```
560     
561     Display remains optional.
562     
563     ---
564     
565     ## WebUiManager
566     
567     Responsibilities:
568     
569     ```text
570     Configuration
571     Calibration
572     Diagnostics
573     Statistics
574     ```
575     
576     The WebUI is the primary configuration interface.
577     
578     ---
579     
580     # 11. Position Sources
581     
582     ## Aircraft Position
583     
584     Provided through:
585     
586     ```text
587     CRSF
588     
589     or
590     
591     MAVLink
592     ```
593     
594     Contains:
595     
596     ```text
597     Latitude
598     Longitude
599     Altitude
600     Speed
601     Heading
602     ```
603     
604     ---
605     
606     ## Tracker Position
607     
608     Provided through:
609     
610     ```text
611     Local GPS
612     ```
613     
614     Provides:
615     
616     ```text
617     Position
618     Satellite Count
619     HDOP
620     Fix Status
621     ```
622     
623     ---
624     
625     # 12. Tracking Modes
626     
627     ## Standard Tracking
628     
629     Current AAT behavior.
630     
631     Requirements:
632     
633     ```text
634     CRSF
635     2 Servos
636     ```
637     
638     ---
639     
640     ## GPS-Aware Tracking
641     
642     Adds:
643     
644     ```text
645     Local GPS
646     Automatic Home
647     ```
648     
649     ---
650     
651     ## Compass Tracking
652     
653     Adds:
654     
655     ```text
656     Absolute Heading
657     Automatic Orientation
658     ```
659     
660     ---
661     
662     ## Continuous 360° Tracking
663     
664     Adds:
665     
666     ```text
667     Shortest Path Logic
668     Continuous Rotation
669     ```
670     
671     ---
672     
673     ## Predictive Tracking
674     
675     Adds:
676     
677     ```text
678     Position Projection
679     Latency Compensation
680     ```
681     
682     ---
683     
684     # 13. WebUI Architecture
685     
686     AAT-Pro shall extend the existing Backpack WebUI framework rather than introducing a separate implementation. ?3-ed8fcb?
687     
688     ---
689     
690     ## Status Page
691     
692     Displays:
693     
694     ```text
695     GPS Status
696     Compass Status
697     Distance
698     Bearing
699     Elevation
700     Telemetry Status
701     ```
702     
703     ---
704     
705     ## Configuration Page
706     
707     Provides:
708     
709     ```text
710     Home Source
711     Telemetry Source
712     Heading Source
713     Pan Mode
714     ```
715     
716     ---
717     
718     ## Servo Page
719     
720     Provides:
721     
722     ```text
723     Limits
724     Direction
725     PID
726     Deadband
727     Motion Limits
728     ```
729     
730     ---
731     
732     ## GPS Page
733     
734     Displays:
735     
736     ```text
737     Tracker GPS
738     Aircraft GPS
739     Home Position
740     GPS Health
741     ```
742     
743     ---
744     
745     ## Compass Page
746     
747     Provides:
748     
749     ```text
750     Calibration
751     Heading
752     Health Status
753     ```
754     
755     ---
756     
757     ## Prediction Page
758     
759     Displays:
760     
761     ```text
762     Prediction Mode
763     Latency Estimate
764     Prediction Health
765     ```
766     
767     ---
768     
769     ## Diagnostics Page
770     
771     Displays:
772     
773     ```text
774     Telemetry Rate
775     Packet Loss
776     Servo Status
777     Prediction Error
778     System Health
779     ```
780     
781     ---
782     
783     # 14. Configuration Storage
784     
785     Configuration shall be stored using Backpack-native persistence mechanisms.
786     
787     Storage implementation shall be reused whenever possible.
788     
789     Examples:
790     
791     ```text
792     Preferences
793     
794     or
795     
796     NVS
797     
798     or
799     
800     Existing Backpack Configuration Storage
801     ```
802     
803     Final implementation shall follow the existing Backpack architecture.
804     
805     ---
806     
807     # 15. Safety Architecture
808     
809     ## Home Validation
810     
811     Requirements:
812     
813     ```text
814     3D Fix
815     
816     Satellites >= 10
817     
818     HDOP <= 1.5
819     
820     Stable Position >= 30 seconds
821     ```
822     
823     ---
824     
825     ## GPS Jump Rejection
826     
827     Purpose:
828     
829     ```text
830     Reject invalid GPS spikes
831     ```
832     
833     ---
834     
835     ## Compass Validation
836     
837     Purpose:
838     
839     ```text
840     Reject invalid heading data
841     ```
842     
843     ---
844     
845     ## Zenith Protection
846     
847     Purpose:
848     
849     ```text
850     Avoid unstable bearing calculations
851     when the aircraft is overhead.
852     ```
853     
854     ---
855     
856     ## Telemetry Watchdog
857     
858     Timeout:
859     
860     ```text
861     3 seconds
862     ```
863     
864     Actions:
865     
866     ```text
867     Hold
868     
869     Park
870     
871     Return Home Direction
872     ```
873     
874     ---
875     
876     ## Servo Fault Detection
877     
878     Purpose:
879     
880     ```text
881     Detect blocked servos
882     or abnormal tracking behavior.
883     ```
884     
885     ---
886     
887     # 16. Performance Objectives
888     
889     Tracking Loop:
890     
891     ```text
892     10 Hz minimum
893     
894     20 Hz target
895     ```
896     
897     ---
898     
899     Servo Updates:
900     
901     ```text
902     50 Hz target
903     ```
904     
905     ---
906     
907     WebUI:
908     
909     ```text
910     Must never degrade
911     tracking performance.
912     ```
913     
914     ---
915     
916     # 17. Backward Compatibility Rules
917     
918     Rule 1
919     
920     ```text
921     Current AAT behavior remains unchanged.
922     ```
923     
924     ---
925     
926     Rule 2
927     
928     ```text
929     AAT Classic remains supported.
930     ```
931     
932     ---
933     
934     Rule 3
935     
936     ```text
937     All advanced features are optional.
938     ```
939     
940     ---
941     
942     Rule 4
943     
944     ```text
945     Missing optional hardware
946     shall never prevent basic tracking.
947     ```
948     
949     Examples:
950     
951     ```text
952     No Compass
953     ? Manual Heading
954     ```
955     
956     ```text
957     No GPS
958     ? Telemetry Home
959     ```
960     
961     ```text
962     No OLED
963     ? WebUI Only
964     ```
965     
966     ---
967     
968     Rule 5
969     
970     ```text
971     Existing Backpack targets
972     shall not require modification.
973     ```
974     
975     ---
976     
977     # 18. Long-Term Vision
978     
979     AAT-Pro aims to combine:
980     
981     - Native ExpressLRS integration
982     - Existing Backpack ecosystem
983     - Continuous rotation tracking concepts
984     - Local GPS awareness
985     - Compass-based orientation
986     - Predictive tracking
987     - Automatic latency estimation
988     - Advanced diagnostics
989     - MAVLink support
990     
991     while remaining fully aligned with Backpack architecture and retaining compatibility with existing AAT deployments.
992     
993     ---
994     
995     # 19. Architectural Success Criteria
996     
997     The architecture shall be considered successful when:
998     
999     - AAT Classic remains unaffected.
1000     - New targets integrate using Backpack infrastructure.
1001     - No parallel configuration system exists.
1002     - Most functionality is configurable through WebUI.
1003     - Advanced hardware is optional.
1004     - Features can be submitted upstream as independent pull requests.
1005     - The system scales from ESP8266 AAT Classic to ESP32-S3 AAT-Pro Full.

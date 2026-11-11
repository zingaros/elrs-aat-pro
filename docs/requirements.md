1     # Requirements Specification
2     ## AAT-Pro
3     ### Advanced ExpressLRS Backpack Antenna Tracker
4     
5     Version: 0.2 Draft
6     
7     Status: Requirements Definition
8     
9     Related Documents:
10     
11     - architecture.md
12     - tracking_algorithm.md
13     - backlog.md
14     - hardware_profiles.md
15     - board_profiles.md
16     - pinout.md
17     
18     ---
19     
20     # 1. Purpose
21     
22     This document defines the functional and non-functional requirements for AAT-Pro.
23     
24     AAT-Pro is an advanced antenna tracker implemented as an extension of the ExpressLRS Backpack ecosystem.
25     
26     AAT-Pro shall not become a parallel ecosystem.
27     
28     The project shall preserve compatibility with existing AAT deployments while enabling more advanced tracking capabilities on ESP32-based hardware.
29     
30     ---
31     
32     # 2. Project Constraints
33     
34     ## REQ-CONST-001
35     
36     AAT-Pro shall be implemented as an ExpressLRS Backpack target.
37     
38     Priority: MUST
39     
40     ---
41     
42     ## REQ-CONST-002
43     
44     AAT-Pro shall reuse existing Backpack infrastructure whenever possible.
45     
46     Priority: MUST
47     
48     Examples:
49     
50     - WebUI framework
51     - Target registration
52     - Flashing workflow
53     - Configuration storage
54     - Telemetry forwarding
55     - Build system
56     
57     ---
58     
59     ## REQ-CONST-003
60     
61     AAT-Pro shall not introduce a standalone firmware ecosystem.
62     
63     Priority: MUST
64     
65     ---
66     
67     ## REQ-CONST-004
68     
69     AAT-Pro shall remain suitable for upstream integration.
70     
71     Priority: MUST
72     
73     ---
74     
75     ## REQ-CONST-005
76     
77     All advanced functionality shall remain modular.
78     
79     Priority: MUST
80     
81     ---
82     
83     # 3. Product Variants
84     
85     ## REQ-PROD-001
86     
87     The firmware shall support the following target families:
88     
89     ```text
90     AAT_CLASSIC
91     
92     AAT_PRO_ESP32
93     
94     AAT_PRO_ESP32_S3
95     ```
96     
97     Priority: MUST
98     
99     ---
100     
101     ## REQ-PROD-002
102     
103     AAT Classic functionality shall remain unchanged.
104     
105     Priority: MUST
106     
107     ---
108     
109     ## REQ-PROD-003
110     
111     AAT-Pro functionality shall be additive.
112     
113     Priority: MUST
114     
115     ---
116     
117     ## REQ-PROD-004
118     
119     Advanced features shall be disabled by default.
120     
121     Priority: MUST
122     
123     ---
124     
125     ## REQ-PROD-005
126     
127     Users upgrading existing AAT installations shall experience no behavior changes unless advanced features are explicitly enabled.
128     
129     Priority: MUST
130     
131     ---
132     
133     # 4. Capability Requirements
134     
135     ## REQ-CAP-001
136     
137     All advanced features shall be capability-driven.
138     
139     Priority: MUST
140     
141     ---
142     
143     ## REQ-CAP-002
144     
145     Tracking logic shall not directly depend on board names.
146     
147     Priority: MUST
148     
149     Preferred:
150     
151     ```cpp
152     if(capability.compass)
153     ```
154     
155     Forbidden:
156     
157     ```cpp
158     if(board == ESP32S3)
159     ```
160     
161     ---
162     
163     ## REQ-CAP-003
164     
165     Capabilities shall be determined using:
166     
167     ```text
168     Target Definition
169     +
170     Runtime Detection
171     +
172     User Configuration
173     ```
174     
175     Priority: MUST
176     
177     ---
178     
179     ## REQ-CAP-004
180     
181     Missing optional capabilities shall not prevent operation.
182     
183     Priority: MUST
184     
185     ---
186     
187     ## REQ-CAP-005
188     
189     All optional capabilities shall support graceful fallback behavior.
190     
191     Priority: MUST
192     
193     ---
194     
195     # 5. Target System Requirements
196     
197     ## REQ-TARGET-001
198     
199     AAT-Pro targets shall be registered through the Backpack target system.
200     
201     Priority: MUST
202     
203     ---
204     
205     ## REQ-TARGET-002
206     
207     Target definitions shall reuse Backpack target registration mechanisms.
208     
209     Priority: MUST
210     
211     ---
212     
213     ## REQ-TARGET-003
214     
215     Target definitions shall specify:
216     
217     ```text
218     Platform
219     Capabilities
220     Firmware
221     Upload Methods
222     Default Pin Assignments
223     ```
224     
225     Priority: MUST
226     
227     ---
228     
229     ## REQ-TARGET-004
230     
231     Target additions shall not require modifications to existing AAT Classic targets.
232     
233     Priority: MUST
234     
235     ---
236     
237     # 6. Telemetry Requirements
238     
239     ## REQ-TLM-001
240     
241     The system shall support CRSF telemetry.
242     
243     Priority: MUST
244     
245     ---
246     
247     ## REQ-TLM-002
248     
249     The system shall support Backpack telemetry forwarding.
250     
251     Priority: MUST
252     
253     ---
254     
255     ## REQ-TLM-003
256     
257     The system shall support MAVLink telemetry.
258     
259     Priority: SHOULD
260     
261     ---
262     
263     ## REQ-TLM-004
264     
265     The system shall automatically detect telemetry loss.
266     
267     Priority: MUST
268     
269     Acceptance Criteria:
270     
271     ```text
272     3 seconds without telemetry
273     ? Telemetry Loss Event
274     ```
275     
276     ---
277     
278     ## REQ-TLM-005
279     
280     The system shall report telemetry update rate.
281     
282     Priority: SHOULD
283     
284     ---
285     
286     ## REQ-TLM-006
287     
288     The system shall report packet loss statistics.
289     
290     Priority: SHOULD
291     
292     ---
293     
294     # 7. GPS Requirements
295     
296     ## REQ-GPS-001
297     
298     The system shall support aircraft GPS data.
299     
300     Priority: MUST
301     
302     ---
303     
304     ## REQ-GPS-002
305     
306     The system shall support an optional tracker GPS.
307     
308     Priority: MUST
309     
310     ---
311     
312     ## REQ-GPS-003
313     
314     The system shall support automatic home acquisition.
315     
316     Priority: MUST
317     
318     ---
319     
320     ## REQ-GPS-004
321     
322     Home acquisition shall require:
323     
324     ```text
325     3D Fix
326     Satellite Count >= 10
327     HDOP <= 1.5
328     Stable Position >= 30 seconds
329     ```
330     
331     Priority: MUST
332     
333     ---
334     
335     ## REQ-GPS-005
336     
337     The system shall reject implausible GPS jumps.
338     
339     Priority: MUST
340     
341     ---
342     
343     ## REQ-GPS-006
344     
345     Tracker GPS failure shall not prevent operation.
346     
347     Priority: MUST
348     
349     Fallback:
350     
351     ```text
352     Telemetry Home
353     ```
354     
355     ---
356     
357     # 8. Compass Requirements
358     
359     ## REQ-COMP-001
360     
361     The system shall support an optional compass.
362     
363     Priority: MUST
364     
365     ---
366     
367     ## REQ-COMP-002
368     
369     The following sensors should be supported:
370     
371     ```text
372     QMC5883L
373     LIS3MDL
374     BMM150
375     ```
376     
377     Priority: SHOULD
378     
379     ---
380     
381     ## REQ-COMP-003
382     
383     Compass calibration shall be supported.
384     
385     Priority: MUST
386     
387     ---
388     
389     ## REQ-COMP-004
390     
391     Compass health shall be monitored.
392     
393     Priority: MUST
394     
395     ---
396     
397     ## REQ-COMP-005
398     
399     Compass failure shall not stop tracking.
400     
401     Priority: MUST
402     
403     Fallback:
404     
405     ```text
406     Manual Heading Mode
407     ```
408     
409     ---
410     
411     # 9. Navigation Requirements
412     
413     ## REQ-NAV-001
414     
415     The system shall calculate:
416     
417     ```text
418     Distance
419     Bearing
420     Elevation
421     ```
422     
423     Priority: MUST
424     
425     ---
426     
427     ## REQ-NAV-002
428     
429     Geodetic coordinate calculations shall be used.
430     
431     Priority: MUST
432     
433     ---
434     
435     ## REQ-NAV-003
436     
437     Elevation shall support relative altitude calculations.
438     
439     Priority: MUST
440     
441     ---
442     
443     # 10. Pan/Tilt Requirements
444     
445     ## REQ-PAN-001
446     
447     Standard servo mode shall be supported.
448     
449     Priority: MUST
450     
451     ---
452     
453     ## REQ-PAN-002
454     
455     Continuous 360° pan mode shall be supported.
456     
457     Priority: MUST
458     
459     ---
460     
461     ## REQ-PAN-003
462     
463     Continuous pan tracking shall use shortest-path calculations.
464     
465     Priority: MUST
466     
467     ---
468     
469     ## REQ-PAN-004
470     
471     Tilt servos up to 180° shall be supported.
472     
473     Priority: SHOULD
474     
475     ---
476     
477     ## REQ-PAN-005
478     
479     Maximum rotation speed shall be configurable.
480     
481     Priority: MUST
482     
483     ---
484     
485     ## REQ-PAN-006
486     
487     Maximum angular acceleration shall be configurable.
488     
489     Priority: SHOULD
490     
491     ---
492     
493     # 11. Filtering Requirements
494     
495     ## REQ-FLT-001
496     
497     GPS filtering shall be supported.
498     
499     Priority: MUST
500     
501     ---
502     
503     ## REQ-FLT-002
504     
505     EMA filtering shall be supported.
506     
507     Priority: MUST
508     
509     ---
510     
511     ## REQ-FLT-003
512     
513     Servo deadband shall be supported.
514     
515     Priority: MUST
516     
517     ---
518     
519     ## REQ-FLT-004
520     
521     Deadband shall be configurable.
522     
523     Priority: MUST
524     
525     ---
526     
527     ## REQ-FLT-005
528     
529     Distance-adaptive deadband should be supported.
530     
531     Priority: SHOULD
532     
533     ---
534     
535     ## REQ-FLT-006
536     
537     Tilt easing should be supported.
538     
539     Priority: SHOULD
540     
541     ---
542     
543     # 12. PID Requirements
544     
545     ## REQ-PID-001
546     
547     PID control shall be supported.
548     
549     Priority: SHOULD
550     
551     ---
552     
553     ## REQ-PID-002
554     
555     Independent PID controllers shall be available for:
556     
557     ```text
558     Pan
559     Tilt
560     ```
561     
562     Priority: SHOULD
563     
564     ---
565     
566     ## REQ-PID-003
567     
568     PID shall remain disabled by default.
569     
570     Priority: MUST
571     
572     ---
573     
574     # 13. Prediction Requirements
575     
576     ## REQ-PRED-001
577     
578     Prediction shall be optional.
579     
580     Priority: MUST
581     
582     ---
583     
584     ## REQ-PRED-002
585     
586     Linear prediction shall be supported.
587     
588     Priority: MUST
589     
590     ---
591     
592     ## REQ-PRED-003
593     
594     Adaptive prediction shall be supported.
595     
596     Priority: SHOULD
597     
598     ---
599     
600     ## REQ-PRED-004
601     
602     The system shall support short-term tracking continuation during telemetry outages.
603     
604     Priority: SHOULD
605     
606     ---
607     
608     ## REQ-PRED-005
609     
610     Prediction failure shall automatically revert to standard tracking.
611     
612     Priority: MUST
613     
614     ---
615     
616     # 14. Latency Compensation Requirements
617     
618     ## REQ-LAT-001
619     
620     Manual latency compensation shall be supported.
621     
622     Priority: MUST
623     
624     ---
625     
626     ## REQ-LAT-002
627     
628     Automatic latency compensation shall be supported.
629     
630     Priority: SHOULD
631     
632     ---
633     
634     ## REQ-LAT-003
635     
636     Latency estimation range:
637     
638     ```text
639     200 ms
640     to
641     1000 ms
642     ```
643     
644     Priority: MUST
645     
646     ---
647     
648     ## REQ-LAT-004
649     
650     Prediction quality shall be monitored continuously.
651     
652     Priority: SHOULD
653     
654     ---
655     
656     ## REQ-LAT-005
657     
658     The system shall automatically reduce prediction aggressiveness if prediction quality degrades.
659     
660     Priority: SHOULD
661     
662     ---
663     
664     # 15. Safety Requirements
665     
666     ## REQ-SAFE-001
667     
668     Telemetry loss shall be detected.
669     
670     Priority: MUST
671     
672     ---
673     
674     ## REQ-SAFE-002
675     
676     Configurable telemetry-loss behavior shall be supported.
677     
678     Priority: MUST
679     
680     Available Modes:
681     
682     ```text
683     Hold
684     Park
685     Return Home Direction
686     ```
687     
688     ---
689     
690     ## REQ-SAFE-003
691     
692     Zenith protection shall be implemented.
693     
694     Priority: MUST
695     
696     ---
697     
698     ## REQ-SAFE-004
699     
700     Servo health monitoring should be supported.
701     
702     Priority: SHOULD
703     
704     ---
705     
706     ## REQ-SAFE-005
707     
708     Servo stall detection should be supported.
709     
710     Priority: SHOULD
711     
712     ---
713     
714     ## REQ-SAFE-006
715     
716     Startup self-tests should be supported.
717     
718     Priority: SHOULD
719     
720     ---
721     
722     # 16. Configuration Storage Requirements
723     
724     ## REQ-CFG-001
725     
726     Configuration shall use Backpack-native persistence mechanisms.
727     
728     Priority: MUST
729     
730     ---
731     
732     ## REQ-CFG-002
733     
734     No independent storage system shall be introduced.
735     
736     Priority: MUST
737     
738     ---
739     
740     ## REQ-CFG-003
741     
742     Configuration migration shall not require manual intervention.
743     
744     Priority: SHOULD
745     
746     ---
747     
748     # 17. User Interface Requirements
749     
750     ## REQ-UI-001
751     
752     All functionality shall remain usable without an OLED display.
753     
754     Priority: MUST
755     
756     ---
757     
758     ## REQ-UI-002
759     
760     Operation without display shall be supported.
761     
762     Priority: MUST
763     
764     ---
765     
766     ## REQ-UI-003
767     
768     Operation with OLED display shall be supported.
769     
770     Priority: MUST
771     
772     ---
773     
774     ## REQ-UI-004
775     
776     Single-button operation shall be supported.
777     
778     Priority: MUST
779     
780     ---
781     
782     ## REQ-UI-005
783     
784     Additional buttons shall remain optional.
785     
786     Priority: SHOULD
787     
788     ---
789     
790     # 18. WebUI Requirements
791     
792     ## REQ-WEB-001
793     
794     AAT-Pro shall extend the existing Backpack WebUI infrastructure.
795     
796     Priority: MUST
797     
798     ---
799     
800     ## REQ-WEB-002
801     
802     No standalone WebUI framework shall be created.
803     
804     Priority: MUST
805     
806     ---
807     
808     ## REQ-WEB-003
809     
810     AAT-Pro pages shall integrate into existing Backpack navigation.
811     
812     Priority: SHOULD
813     
814     ---
815     
816     ## REQ-WEB-004
817     
818     A Status page shall be provided.
819     
820     Priority: MUST
821     
822     ---
823     
824     ## REQ-WEB-005
825     
826     A Configuration page shall be provided.
827     
828     Priority: MUST
829     
830     ---
831     
832     ## REQ-WEB-006
833     
834     A Servo page shall be provided.
835     
836     Priority: MUST
837     
838     ---
839     
840     ## REQ-WEB-007
841     
842     A Diagnostics page shall be provided.
843     
844     Priority: MUST
845     
846     ---
847     
848     ## REQ-WEB-008
849     
850     A GPS page should be provided.
851     
852     Priority: SHOULD
853     
854     ---
855     
856     ## REQ-WEB-009
857     
858     A Compass page should be provided.
859     
860     Priority: SHOULD
861     
862     ---
863     
864     ## REQ-WEB-010
865     
866     A Prediction page should be provided.
867     
868     Priority: SHOULD
869     
870     ---
871     
872     # 19. Diagnostics Requirements
873     
874     ## REQ-DIAG-001
875     
876     The system shall expose telemetry statistics.
877     
878     Priority: SHOULD
879     
880     ---
881     
882     ## REQ-DIAG-002
883     
884     The system shall expose GPS health information.
885     
886     Priority: SHOULD
887     
888     ---
889     
890     ## REQ-DIAG-003
891     
892     The system shall expose compass health information.
893     
894     Priority: SHOULD
895     
896     ---
897     
898     ## REQ-DIAG-004
899     
900     The system shall expose prediction metrics.
901     
902     Priority: SHOULD
903     
904     ---
905     
906     ## REQ-DIAG-005
907     
908     Debug logging should be supported.
909     
910     Priority: SHOULD
911     
912     ---
913     
914     # 20. Performance Requirements
915     
916     ## REQ-PERF-001
917     
918     Tracking calculations shall never block telemetry processing.
919     
920     Priority: MUST
921     
922     ---
923     
924     ## REQ-PERF-002
925     
926     The tracking loop shall execute deterministically.
927     
928     Priority: MUST
929     
930     ---
931     
932     ## REQ-PERF-003
933     
934     WebUI activity shall not degrade tracking performance.
935     
936     Priority: MUST
937     
938     ---
939     
940     ## REQ-PERF-004
941     
942     The architecture shall remain compatible with ESP32-class hardware.
943     
944     Priority: MUST
945     
946     ---
947     
948     # 21. Compatibility Requirements
949     
950     ## REQ-COMPAT-001
951     
952     AAT Classic users shall experience no functional change.
953     
954     Priority: MUST
955     
956     ---
957     
958     ## REQ-COMPAT-002
959     
960     Existing Backpack targets shall remain supported.
961     
962     Priority: MUST
963     
964     ---
965     
966     ## REQ-COMPAT-003
967     
968     Optional hardware failures shall not prevent operation.
969     
970     Priority: MUST
971     
972     ---
973     
974     ## REQ-COMPAT-004
975     
976     Current Backpack targets shall compile without modification.
977     
978     Priority: MUST
979     
980     ---
981     
982     ## REQ-COMPAT-005
983     
984     Disabling AAT-Pro features shall restore baseline behavior.
985     
986     Priority: MUST
987     
988     ---
989     
990     ## REQ-COMPAT-006
991     
992     The firmware shall compile cleanly when optional hardware is absent.
993     
994     Priority: MUST
995     
996     ---
997     
998     # 22. Upstream Contribution Requirements
999     
1000     ## REQ-UPSTREAM-001
1001     
1002     Features shall be divisible into independent pull requests.
1003     
1004     Priority: MUST
1005     
1006     ---
1007     
1008     ## REQ-UPSTREAM-002
1009     
1010     Each pull request shall provide standalone value.
1011     
1012     Priority: MUST
1013     
1014     ---
1015     
1016     ## REQ-UPSTREAM-003
1017     
1018     Each feature shall degrade gracefully when disabled.
1019     
1020     Priority: MUST
1021     
1022     ---
1023     
1024     ## REQ-UPSTREAM-004
1025     
1026     No pull request shall require all AAT-Pro functionality to be merged simultaneously.
1027     
1028     Priority: MUST
1029     
1030     ---
1031     
1032     # 23. Release Roadmap
1033     
1034     ## v0.1
1035     
1036     ```text
1037     Refactoring Foundation
1038     ```
1039     
1040     ---
1041     
1042     ## v0.2
1043     
1044     ```text
1045     Target Definition Layer
1046     ```
1047     
1048     ---
1049     
1050     ## v0.3
1051     
1052     ```text
1053     Local GPS
1054     ```
1055     
1056     ---
1057     
1058     ## v0.4
1059     
1060     ```text
1061     Compass Support
1062     ```
1063     
1064     ---
1065     
1066     ## v0.5
1067     
1068     ```text
1069     Continuous 360° Tracking
1070     ```
1071     
1072     ---
1073     
1074     ## v0.6
1075     
1076     ```text
1077     WebUI Extensions
1078     ```
1079     
1080     ---
1081     
1082     ## v0.7
1083     
1084     ```text
1085     Filtering
1086     ```
1087     
1088     ---
1089     
1090     ## v0.8
1091     
1092     ```text
1093     PID Support
1094     ```
1095     
1096     ---
1097     
1098     ## v0.9
1099     
1100     ```text
1101     Prediction Engine
1102     ```
1103     
1104     ---
1105     
1106     ## v1.0
1107     
1108     ```text
1109     Diagnostics
1110     ```
1111     
1112     ---
1113     
1114     ## v1.1
1115     
1116     ```text
1117     Automatic Latency Compensation
1118     ```
1119     
1120     ---
1121     
1122     ## v1.2
1123     
1124     ```text
1125     MAVLink Support
1126     ```
1127     
1128     ---
1129     
1130     # 24. Success Criteria
1131     
1132     AAT-Pro shall be considered successful when:
1133     
1134     - AAT Classic remains unaffected.
1135     - New Backpack targets integrate cleanly.
1136     - No parallel ecosystem is created.
1137     - Configuration is primarily WebUI-based.
1138     - Advanced hardware remains optional.
1139     - Features are mergeable as independent pull requests.
1140     - The architecture scales from ESP8266 AAT Classic to ESP32-S3 AAT-Pro Full.

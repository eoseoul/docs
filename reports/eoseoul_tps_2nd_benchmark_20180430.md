## EOSeoul Report TPS(24th April) and BlockOne Lab Test(26th April)
 
On 24th April, EOSeoul sent our BMT TPS results and related questions to BlockOne.

* https://steemit.com/en/@eoseoul/bmt-eosio-tps-results-by-eoseoul 

BlockOne replied by email / Steemit /Telegram and shared the test methodology on how they obtained TPS data. 

* Instructions : https://gist.github.com/spoonincode/fca5658326837b76fd744d39b2a25b4e
* github issue : https://github.com/EOSIO/eos/issues/2078 
* Video demo : https://vimeo.com/266585781 

BlockOne’s methodology involves using the binaryen interpreter between 1 block producing node and 1 full node. In other words, it is a lab test that tests only the engine performance of EOS. It excludes various factors that could affect TPS in a real world environment as it only calculates the core functionality of transactions via P2P. It is very much like an official mpg (miles per gallon) stamp for automobiles.

EOSeoul’s TPS methodology was slightly different from BlockOne’s lab test. We adopted the cloud environment EOS settings used by most BP candidates at the moment. Most BP nodes have active HTTP plugin that receive transactions. We performed a stress test by sending enough queries to see how much performance degradation occurs – this is the stress benchmark. Our test is based on early Mainnet environment and thus our results are lower than that of BlockOne’s. It is like an observed mpg (miles per gallon) for automobiles.


## BlockOne test verification and replication

EOSeoul performed BlockOne’s lab test method and recorded 1200+ TPS  with a intel i7-6700  3.4GHz CPU. Similarly, we recorded 2000+TPS with JIT active. Below is the performance result when we perform the test in a cloud environment.

### BlockOne Test method in a cloud environment
#### Test Environment
* Build : DAWN-v3.0.0 (d9ad8eec)
* AWS EC2 m5.2xlarge 
  * CPU : Intel Xeon Platinum 8175M 2.5Ghz * 8 Core
  * Mem : 32G
  * Disk : EBS 120GB SSD (3000 IOPS Fix)
* AWS EC2 C5.2xlarge 
  * CPU : Intel Xeon Platinum 8124M 3.0Ghz * 8 Core
  * Mem : 16G
  * Disk : EBS 120GB SSD (3000 IOPS Fix)
#### Test explanation
* txn_test_gen Settings
  * generation cycle(ms) x Transaction Count(count per interval)
  * 20x20 : 20 transactions per 20ms
  * 50x60 : 60 transactions per 50ms
#### Benchmark1. M5.2xlarge - EC2 1 : FN 1 + BPN 1 
* 20x20 (1000TPS) : Recorded 1000 TPS. However, it feels like transactions start queuing up after a while. Feels unstable. BP node CPU : 42%, Full node CPU : 82%
* 22x20 (Approx 900TPS) : Fairly stable. BP node : 35%, Full node : approx 78%
* 50x60 (1200TPS) : Halts after CPU reaches 100%. Cannot be processed.
* 60X58 (966TPS) : Unstable after a while. Recorded 966 TPS. BP node : 40%, Full node : 79%

#### Benchmark 2. C5.2xlarge - EC2 1 : FN 1 + BPN 1
* 20x20 : 1000 TPS processed. Very stable. BP node CPU 30% , Full node 65%
* 20x22 : 1100 TPS processed. Very stable. BP node CPU 34~35%, Full node 75%
* 20x24 : 1200 TPS processed. Fairly stable. BP node CPU 38%, Full node 80%
* 20x26 : 1300 TPS cannt be achieved. Approx 1000 TPS. BP Ndoe CPU : 43%, Full node : 83% 
* 50x60 : 1200 TPS. Same as above.
* 50x66 : 1320 TPS should be achievable but failed.
 
#### Benchmark 3. C5 EC2 2 machines : FN1 + BPN1
* 20x20 : 1000 TPS processed. Very stable. BP node CPU 30% , Full node 60%
* 20x22 : 1100 TPS processed. Very stable. BP node CPU 36%, Full node 70%
* 20x24 : 1200 TPS processed. Fairly stable but feels like there is a little bit of queuing up. BP node CPU 38%, Full node 78~80%
* 20x26 : 1300 TPS processed. Fairly stable. Actually feels better than 1200 TPS. BP node 42%, Full node 82~84% 
* 20x28 : 1400 TPS should be possible but in reality records 1000 TPS. CPU usage is volatile.

#### Benchmark 4. C5 EC2 3 machines: FN2 + BPN1
* 20x20 : 1000 TPS. Very stable. BP node 33~38% , Full node1 58~64%, Full node2 52~56% 
* 20x22 : 1100TPS. Very stable. BP node 38~40%, Full node1 68~72%, Full node2 58~64%
* 20x24 : 1200TPS. Very stable. BP node 40~42%, Full node1 74~80%, Full node2 60~70%
* 20x26 : 1300TPS. Not stable. Approx 1000~1100 TPS, BP, Full node shows volatile CPU usage. At certain points of time TPS records 966 but then drops down to 99. Obviously CPU usage is low when TPS is low.
* 20x28 : 1400 TPS Unable to be processed. BMT stopped due to error. 
* 20x12x2 : 600+600 TPS. Stable. Slight queuing maybe due to broadcast time or network delivery time from using 2 nodes recording 600TPS each. BP node : 40~44%, Full node1 : 68~74%, Full node2 : 76~80%, Full node2 records slightly higher CPU usage. 
* 20x14x2 : 700+700TPS. Records 1400 TPS at a certain moment but error occurs on one of the 2 nodes. Error occurs as CPU allocation is not possible. BP node 58~64%, Full node : max 98%+, test terminated due to error so CPU usage doesn’t mean much
* 20x20+20x6 : 1000 + 300 TPS. Unstable and TEST Job (300TPS) terminated. CPU results same as no. 7 

### Activate JIT in cloud environment and verify BlockOne’s test methodology

#### Benchmark 5. C5 EC2 2 machines : FN1 + BPN1 (Using WAVM ~ JIT)
* 20x20(1000 TPS) : Reach 1000TPS very comfortably. CPU usage BP node 30~34%, Full node approx 33~37%
* 20x30(1500 TPS) : Reach 1500 TPS very comfortably. BP node 50%, Full node approx. 55%
* 20x36(1800 TPS) : Stable 1800 TPS. BP node 56~60%, Full node approx. 66~70%
* 20x38(1900 TPS) : Stable 1900 TPS. BP node 58~62%, Full node approx. 68~72%
* 20x40(2000 TPS) : Fails immediately. Looks like overload. Need testing on a servicer with a higher CPU Clock or need further adjustments. 

### Comment on Test results

* 1000 TPS(Worst case) mentioned in Dawn 3.0 by Dan Larimer could be achieved. However, this result was an engine performance test and would be different in a real environment
* In early days of mainnet, it seems optimal to use servers with high performance CPU’s. (seen through Benchmark 1 and Benchmark 2)
* We could see that there is a slight performance gap after separating full node and BP node. It seems that it is correct to separate the two. However, we do not expect there to be massive differences.
* It is rational to run the Mainnet stress test with BP node and Full node set up separately. The current architecture of the TESTNET where BP node takes on the role of Full node isn’t the most optimal structure from a performance perspective.
* There is a decline in performance as Full nodes are added. One Full node and one BP node gave a stable 1300 TPS whereas two Full nodes and One BP node showed unstable performance.
* We expect declining performance as BP nodes are added because communication between the BP nodes will use up resources.
* We believe performance decline is due to transaction broadcasts between nodes. Thus, the more nodes there are, the poorer the performance will become.
* JIT activation can result in better performance. As in Dawn 3.0, optimal performance can be achieved by using binaryen in early contracts together with JIT

## Thank you

EOSeoul has received many feedbacks since releasing the EOSIO benchmark test results. We very much appreciate your feedbacks.
 
First of all, we thank Daniel Larimer and developers at BlockOne. We apologize if the benchmark report on 24th April made you unpleasant. Our intention was to test EOSIO’s performance in an average environment. We thought that txn_test_gen plugin was rather simple from a stress text perspective and assumed that BlockOne obtained results with a more complex benchmark scenario in its Dawn 3.0 document released on 6th April.

We do see EOSIO’s technological development through the logs on github. We thank you for your kind reply.

We also thank the various feedbacks from EOS BP’s. The discussions and feedbacks will allow us to perform reliable tests in the future. In particular we thank EOS Cannon for sharing their lab test results (BlockOne) with us. We will continue to share test results in order to launch a stable Mainnet.


## We suggest sharing the stress benchmark script and the results
 
EOS community wishes for same methodology to be used to test the performance of BP’s nodes. EOSeoul has released the stress benchmark script on github.

* https://github.com/eoseoul/scripts/tree/master/bmt_client 

Every block producer can test their performance using this script. We recommend BP’s to suggest scripts and share performance results with the community. EOSeoul will contribute to the community by offering a standardized performance test.
 
We welcome feedback. Please contact us(EOSeoul) at any time. You can receive EOSeoul’s latest news and engage in technical discussions in our telegram group.

## Channels

* Telegram (English) : http://t.me/eoseoul_en
* Telegram (简体中文) : http://t.me/eoseoul_cn
* Telegram (日本語) : http://t.me/eoseoul_jp
* Telegram (General Talk, 한국어) : https://t.me/eoseoul
* Telegram (Developer Talk, 한국어) : https://t.me/eoseoul_testnet
* Steemit : https://steemit.com/@eoseoul
* Github : https://github.com/eoseoul
* Twitter : https://twitter.com/eoseoul_kor
* Facebook : https://www.facebook.com/EOSeoul.kr
HornetQ bisa dibuat untuk menjalankan live backup pair, dimana dari satu instance EAP akan ada 2 hornetq-server (LIVE dan Backup)

Contoh konfigurasi Domain.xml

   ```
   <subsystem xmlns="urn:jboss:domain:messaging:1.4">
               <hornetq-server name="live">
                   <check-for-live-server>true</check-for-live-server>
                   <failover-on-shutdown>false</failover-on-shutdown>
                   <allow-failback>true</allow-failback>
                   <persistence-enabled>true</persistence-enabled>
                   <cluster-password>1q2w3e4r</cluster-password>
                   <shared-store>false</shared-store>
                   <journal-type>ASYNCIO</journal-type>
                   <journal-min-files>2</journal-min-files>
                   <backup-group-name>${live.messaging.backup.group}</backup-group-name>
                   <paging-directory path="paging-live" relative-to="jboss.server.data.dir"/>
                   <bindings-directory path="bindings-live" relative-to="jboss.server.data.dir"/>
                   <journal-directory path="journal-live" relative-to="jboss.server.data.dir"/>
                   <large-messages-directory path="largemessages-live" relative-to="jboss.server.data.dir"/>

                   <connectors>
                       <netty-connector name="netty" socket-binding="messaging"/>
                       <netty-connector name="netty-throughput" socket-binding="messaging-throughput">
                           <param key="batch-delay" value="50"/>
                       </netty-connector>
                       <in-vm-connector name="in-vm" server-id="${live.messaging.id}"/>
                   </connectors>

                   <acceptors>
                       <netty-acceptor name="netty" socket-binding="messaging"/>
                       <netty-acceptor name="netty-throughput" socket-binding="messaging-throughput">
                           <param key="batch-delay" value="50"/>
                           <param key="direct-deliver" value="false"/>
                       </netty-acceptor>
                       <in-vm-acceptor name="in-vm" server-id="${live.messaging.id}"/>
                   </acceptors>

                   <broadcast-groups>
                       <broadcast-group name="bg-group1">
                           <socket-binding>messaging-group</socket-binding>
                           <broadcast-period>5000</broadcast-period>
                           <connector-ref>
                               netty
                           </connector-ref>
                       </broadcast-group>
                   </broadcast-groups>

                   <discovery-groups>
                       <discovery-group name="dg-group1">
                           <socket-binding>messaging-group</socket-binding>
                           <refresh-timeout>10000</refresh-timeout>
                       </discovery-group>
                   </discovery-groups>

                   <cluster-connections>
                       <cluster-connection name="my-cluster">
                           <address>jms</address>
                           <connector-ref>netty</connector-ref>
                           <discovery-group-ref discovery-group-name="dg-group1"/>
                       </cluster-connection>
                   </cluster-connections>

                   <security-settings>
                       <security-setting match="#">
                           <permission type="send" roles="guest"/>
                           <permission type="consume" roles="guest"/>
                           <permission type="createNonDurableQueue" roles="guest"/>
                           <permission type="deleteNonDurableQueue" roles="guest"/>
                       </security-setting>
                   </security-settings>

                   <address-settings>
                       <address-setting match="#">
                           <dead-letter-address>jms.queue.DLQ</dead-letter-address>
                           <expiry-address>jms.queue.ExpiryQueue</expiry-address>
                           <redelivery-delay>0</redelivery-delay>
                           <max-size-bytes>10485760</max-size-bytes>
                           <address-full-policy>BLOCK</address-full-policy>
                           <message-counter-history-day-limit>10</message-counter-history-day-limit>
                           <redistribution-delay>1000</redistribution-delay>
                       </address-setting>
                   </address-settings>

                   <jms-connection-factories>
                       <connection-factory name="InVmConnectionFactory">
                           <connectors>
                               <connector-ref connector-name="in-vm"/>
                           </connectors>
                           <entries>
                               <entry name="java:/ConnectionFactory"/>
                           </entries>
                       </connection-factory>
                       <connection-factory name="RemoteConnectionFactory">
                           <connectors>
                               <connector-ref connector-name="netty"/>
                           </connectors>
                           <entries>
                               <entry name="java:jboss/exported/jms/RemoteConnectionFactory"/>
                               <entry name="jms/RemoteConnectionFactory"/>
                           </entries>
                           <ha>true</ha>
                           <block-on-acknowledge>true</block-on-acknowledge>
                           <retry-interval>1000</retry-interval>
                           <retry-interval-multiplier>1.0</retry-interval-multiplier>
                           <reconnect-attempts>-1</reconnect-attempts>
                       </connection-factory>
                       <pooled-connection-factory name="hornetq-ra">
                           <transaction mode="xa"/>
                           <connectors>
                               <connector-ref connector-name="in-vm"/>
                           </connectors>
                           <entries>
                               <entry name="java:/JmsXA"/>
                           </entries>
                       </pooled-connection-factory>
                   </jms-connection-factories>
                   <jms-destinations>
                     <jms-topic name="testTopic">
                         <entry name="jms/topic/test"/>
                         <entry name="jboss/exported/jms/topic/test"/>
                     </jms-topic>
                     <jms-queue name="testQueue">
                         <entry name="jms/queue/test"/>
                         <entry name="jboss/exported/jms/queue/test"/>
                     </jms-queue>
                   </jms-destinations>
                 </hornetq-server>
               <hornetq-server name="backup">
                       <persistence-enabled>true</persistence-enabled>
                       <cluster-password>1q2w3e4r</cluster-password>
                       <backup>true</backup>
                       <shared-store>false</shared-store>
                       <failover-on-shutdown>false</failover-on-shutdown>
                       <allow-failback>true</allow-failback>
                       <journal-type>ASYNCIO</journal-type>
                       <journal-min-files>2</journal-min-files>
                       <backup-group-name>${backup.messaging.backup.group}</backup-group-name>
                       <replication-clustername>my-cluster</replication-clustername>
                       <paging-directory path="paging-backup" relative-to="jboss.server.data.dir"/>
                       <bindings-directory path="bindings-backup" relative-to="jboss.server.data.dir"/>
                       <journal-directory path="journal-backup" relative-to="jboss.server.data.dir"/>
                       <large-messages-directory path="largemessages-backup" relative-to="jboss.server.data.dir"/>

                       <connectors>
                           <netty-connector name="netty-backup" socket-binding="messaging-backup"/>
                           <netty-connector name="netty-throughput-backup" socket-binding="messaging-throughput-backup">
                               <param key="batch-delay" value="50"/>
                           </netty-connector>
                           <in-vm-connector name="in-vm" server-id="${backup.messaging.id}"/>
                           <connector name="netty-backup">
                               <factory-class>org.hornetq.core.remoting.impl.netty.NettyConnectorFactory</factory-class>
                               <param key="port" value="5446"/>
                           </connector>
                       </connectors>

                       <acceptors>
                           <netty-acceptor name="netty-backup" socket-binding="messaging-backup"/>
                           <netty-acceptor name="netty-throughput-backup" socket-binding="messaging-throughput-backup">
                               <param key="batch-delay" value="50"/>
                               <param key="direct-deliver" value="false"/>
                           </netty-acceptor>
                           <in-vm-acceptor name="in-vm" server-id="${backup.messaging.id}"/>
                       </acceptors>

                       <broadcast-groups>
                           <broadcast-group name="bg-group1">
                               <socket-binding>messaging-group</socket-binding>
                               <broadcast-period>5000</broadcast-period>
                               <connector-ref>netty-backup</connector-ref>
                           </broadcast-group>
                       </broadcast-groups>

                       <discovery-groups>
                           <discovery-group name="dg-group1">
                               <socket-binding>messaging-group</socket-binding>
                               <refresh-timeout>10000</refresh-timeout>
                           </discovery-group>
                       </discovery-groups>

                       <cluster-connections>
                           <cluster-connection name="my-cluster">
                               <address>jms</address>
                               <connector-ref>netty-backup</connector-ref>
                               <discovery-group-ref discovery-group-name="dg-group1"/>
                           </cluster-connection>
                       </cluster-connections>

                       <security-settings>
                           <security-setting match="#">
                               <permission type="send" roles="guest"/>
                               <permission type="consume" roles="guest"/>
                               <permission type="createNonDurableQueue" roles="guest"/>
                               <permission type="deleteNonDurableQueue" roles="guest"/>
                           </security-setting>
                       </security-settings>
                     </hornetq-server>
             </subsystem>

             .....

             <socket-binding-group name="full-ha-sockets" default-interface="public">
              <!-- Needed for server groups using the 'full-ha' profile  -->
              <socket-binding name="ajp" port="8009"/>
              <socket-binding name="http" port="8080"/>
              <socket-binding name="https" port="8443"/>
              <socket-binding name="jacorb" interface="unsecure" port="3528"/>
              <socket-binding name="jacorb-ssl" interface="unsecure" port="3529"/>
              <socket-binding name="jgroups-mping" port="0" multicast-address="${jboss.default.multicast.address:230.0.0.4}" multicast-port="45700"/>
              <socket-binding name="jgroups-tcp" port="7600"/>
              <socket-binding name="jgroups-tcp-fd" port="57600"/>
              <socket-binding name="jgroups-udp" port="55200" multicast-address="${jboss.default.multicast.address:230.0.0.4}" multicast-port="45688"/>
              <socket-binding name="jgroups-udp-fd" port="54200"/>
              <socket-binding name="messaging" port="5445"/>
              <socket-binding name="messaging-backup" port="5446"/>
              <socket-binding name="messaging-group" port="0" multicast-address="${jboss.messaging.group.address:231.7.7.7}" multicast-port="${jboss.messaging.group.port:9876}"/>
              <socket-binding name="messaging-throughput" port="5455"/>
              <socket-binding name="messaging-throughput-backup" port="5456"/>
              <socket-binding name="modcluster" port="0" multicast-address="224.0.1.105" multicast-port="23364"/>
              <socket-binding name="remoting" port="4447"/>
              <socket-binding name="txn-recovery-environment" port="4712"/>
              <socket-binding name="txn-status-manager" port="4713"/>
              <outbound-socket-binding name="mail-smtp">
                  <remote-destination host="localhost" port="25"/>
              </outbound-socket-binding>
            </socket-binding-group>
   ```



Contoh konfigurasi host node 1

   ```
   <system-properties>
       <property name="live.messaging.id" value="0"/>
       <property name="backup.messaging.id" value="1"/>
       <property name="live.messaging.backup.group" value="backup-group-0"/>
       <property name="backup.messaging.backup.group" value="backup-group-1"/>
   </system-properties>
   ```

Contoh konfigurasi host node 2

   ```
   <system-properties>
    <property name="live.messaging.id" value="1"/>
    <property name="backup.messaging.id" value="0"/>
    <property name="live.messaging.backup.group" value="backup-group-1"/>
    <property name="backup.messaging.backup.group" value="backup-group-0"/>
    </system-properties>
   ```

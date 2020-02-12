## UAVCAN v0 Specification 6.Application-level functions

## Contents
### Application-level functions
 - Node initialization
 - Node status reporting
	 - uavcan.protocol.NodeStatus
 - Node discovery
	 - uavcan.protocol.GetNodeInfo
	 - uavcan.protocol.GetDataTypeInfo
 - Time synchronization
	 - Master
	 - Slave
	 - uavcan.protocol.GlobalTimeSync
 - Node configuration
	 - uavcan.protocol.param.ExecuteOpcode
	 - uavcan.protocol.param.GetSet
	 - uavcan.protocol.param.Empty
	 - uavcan.protocol.param.NumericValue
	 - uavcan.protocol.param.Value
	 - Standard configuration parameters
 - File transfer
	 - Firmware update
	 - uavcan.protocol.file.BeginFirmwareUpdate
	 - uavcan.protocol.file.GetInfo
	 - uavcan.protocol.file.GetDirectoryEntryInfo
	 - uavcan.protocol.file.Delete
	 - uavcan.protocol.file.Read
	 - uavcan.protocol.file.Write
	 - uavcan.protocol.file.EntryType
	 - uavcan.protocol.file.Error
	 - uavcan.protocol.file.Path
 - Debug features
	 - uavcan.protocol.debug.KeyValue
	 - uavcan.protocol.debug.LogMessage
	 - uavcan.protocol.debug.LogLevel
 - Command shell access
	 - uavcan.protocol.AccessCommandShell
 - Panic mode
	 - uavcan.protocol.Panic
 - Dynamic node ID allocation
	 - Allocatee-allocator exchanges
	 - Non-redundant allocator
	 - Redundant allocators

## Application-level functions（应用层函数）
The higher level concepts of UAVCAN are described in this section.  
本节将介绍 UAVCAN 的上层概念。

### Node initialization （节点初始化）
UAVCAN does not require that nodes undergo any specific initialization upon connecting to the bus - a node is free to begin functioning immediately once it is powered up. The only application-level function that every UAVCAN node must support is the periodic broadcasting of the node status message, which is documented next.  
UAVCAN 不要求节点在连接到总线时进行任何特定的初始化 —— 节点在通电后可以立即开始工作。每个 UAVCAN 节点都必须在的应用层支持的功能是：支持定期广播节点状态消息，下面将对此进行说明。

![Alt text](./picture/architecture.png)

### Node status reporting （节点状态报告）
Every UAVCAN node must report its status and presence by broadcasting messages of type uavcan.protocol.NodeStatus. This is the only data structure that UAVCAN nodes are required to support. All other application-level functions are considered optional.  
每个 UAVCAN 节点都必须通过广播 uavcan.protocol.NodeStatus 类型的消息来报告其状态和存在。这是 UAVCAN 节点唯一的必须要支持数据的结构。所有其他应用层函数都被认为是可选的。

Note that the ID of this message contains a long sequence of alternating 0 and 1 values when represented in binary, which facilitates automatic CAN bus bit rate detection.  
注意，当用二进制表示时，此消息的 ID 包含一长串交替的 0 和 1 值，这有助于自动检测 CAN 总线的比特率。被认为是可选的。

The definition of the message is provided below.  
消息的定义如下所示。

__uavcan.protocol.NodeStatus__

Default data type ID: 341

```
#
# Abstract node status information.
#
# Any UAVCAN node is required to publish this message periodically.
#

#
# Publication period may vary within these limits.
# It is NOT recommended to change it at run time.
#
uint16 MAX_BROADCASTING_PERIOD_MS = 1000
uint16 MIN_BROADCASTING_PERIOD_MS = 2

#
# If a node fails to publish this message in this amount of time, it should be considered offline.
#
uint16 OFFLINE_TIMEOUT_MS = 3000

#
# Uptime counter should never overflow.
# Other nodes may detect that a remote node has restarted when this value goes backwards.
#
uint32 uptime_sec

#
# Abstract node health.
#
uint2 HEALTH_OK         = 0     # The node is functioning properly.
uint2 HEALTH_WARNING    = 1     # A critical parameter went out of range or the node encountered a minor failure.
uint2 HEALTH_ERROR      = 2     # The node encountered a major failure.
uint2 HEALTH_CRITICAL   = 3     # The node suffered a fatal malfunction.
uint2 health

#
# Current mode.
#
# Mode OFFLINE can be actually reported by the node to explicitly inform other network
# participants that the sending node is about to shutdown. In this case other nodes will not
# have to wait OFFLINE_TIMEOUT_MS before they detect that the node is no longer available.
#
# Reserved values can be used in future revisions of the specification.
#
uint3 MODE_OPERATIONAL      = 0         # Node is performing its main functions.
uint3 MODE_INITIALIZATION   = 1         # Node is initializing; this mode is entered immediately after startup.
uint3 MODE_MAINTENANCE      = 2         # Node is under maintenance.
uint3 MODE_SOFTWARE_UPDATE  = 3         # Node is in the process of updating its software.
uint3 MODE_OFFLINE          = 7         # Node is no longer available.
uint3 mode

#
# Not used currently, keep zero when publishing, ignore when receiving.
#
uint3 sub_mode

#
# Optional, vendor-specific node status code, e.g. a fault code or a status bitmask.
#
uint16 vendor_specific_status_code
```

### Node discovery
UAVCAN provides mechanisms to obtain the list of all nodes present in the network as well as detailed information about each node:  
UAVCAN 提供了机制用于获取网络中所有节点的列表以及每个节点的详细信息：  

 - Lists of all nodes that are connected to the bus can be created and maintained by listening for node status messages __uavcan.protocol.NodeStatus__.  
 - 可以通过监听节点状态消息__uavcan.protocol.NodeStatus__来创建和维护连接到总线的所有节点的列表。
 <br><br/>
 - Extended information about each node can be requested using the services documented below.  
 - 可以使用下面记录的服务请求关于每个节点的扩展信息。

Note that it is highly recommended to support the service uavcan.protocol.GetNodeInfo in every node, as it is vital for node discovery and identification.  
请注意，强烈建议让每个节点都支持 uavcan.protocol.GetNodeInfo 服务，因为它对于节点发现和标识非常重要。

__uavcan.protocol.GetNodeInfo__

Default data type ID: 1
```
#
# Full node info request.
# Note that all fields of the response section are byte-aligned.
#

---

#
# Current node status
#
NodeStatus status

#
# Version information shall not be changed while the node is running.
#
SoftwareVersion software_version
HardwareVersion hardware_version

#
# Human readable non-empty ASCII node name.
# Node name shall not be changed while the node is running.
# Empty string is not a valid node name.
# Allowed characters are: a-z (lowercase ASCII letters) 0-9 (decimal digits) . (dot) - (dash) _ (underscore).
# Node name is a reversed internet domain name (like Java packages), e.g. "com.manufacturer.project.product".
#
uint8[<=80] name
```

__uavcan.protocol.GetDataTypeInfo__

Default data type ID: 2

```
#
# Get the implementation details of a given data type.
#
# Request is interpreted as follows:
#  - If the field "name' is empty, the fields 'kind' and 'id' will be used to identify the data type.
#  - If the field 'name' is non-empty, it will be used to identify the data type; the
#    fields 'kind' and 'id' will be ignored.
#

uint16 id                   # Ignored if 'name' is non-empty
DataTypeKind kind           # Ignored if 'name' is non-empty

uint8[<=80] name            # Full data type name, e.g. "uavcan.protocol.GetDataTypeInfo"

---

uint64 signature            # Data type signature; valid only if the data type is known (see FLAG_KNOWN)

uint16 id                   # Valid only if the data type is known (see FLAG_KNOWN)
DataTypeKind kind           # Ditto

uint8 FLAG_KNOWN      = 1   # This data type is defined
uint8 FLAG_SUBSCRIBED = 2   # Subscribed to messages of this type
uint8 FLAG_PUBLISHING = 4   # Publishing messages of this type
uint8 FLAG_SERVING    = 8   # Providing service of this type
uint8 flags

uint8[<=80] name            # Full data type name
```

### Time synchronization (时间同步)
UAVCAN supports network-wide precise time synchronization with a resolution of up to 1 CAN bus bit period (i.e., 1 microsecond for 1 Mbps CAN bit rate), assuming that CAN frame timestamping is supported by the hardware. The algorithm can also function in the absence of hardware support for timestamping, although its performance will be degraded.  
UAVCAN 支持网络范围内的精确时间同步，分辨率高达 1 个 CAN 总线 bit 周期（即，1 Mbps 的 CAN bit 速率 时精度时 1 us），假设可以帧时间戳是由硬件支持的。该算法也可以在没有硬件支持的情况下工作，尽管它的性能会下降。

The time synchronization approach is based on the work “Implementing a Distributed High-Resolution Real-Time Clock using the CAN-Bus” (M. Gergeleit and H. Streich). The general idea of the algorithm is to have one or more nodes that periodically broadcast a message of type uavcan.protocol.GlobalTimeSync (definition is provided below) containing the exact timestamp of the previous transmission of this message. A node that performs a periodic broadcast of this message is referred to as a time synchronization master, whereas a node that synchronizes its time with the master is referred to as a time synchronization slave.  
时间同步方法基于“Implementing a Distributed High-Resolution Real-Time Clock using the CAN-Bus”(M. Gergeleit and H. Streich)。该算法的基本思想是让一个或多个节点周期性地广播uavcan.protocol.GlobalTimeSync （定义如下）包含此消息的前一次传输的准确时间戳。执行此消息的定期广播的节点称为时间同步主节点，而将其时间与主节点同步的节点称为时间同步从节点。

Note that this algorithm only allows to precisely estimate the phase difference between the given slave and the master it is synchronized with. UAVCAN does not define the algorithm for clock speed/phase adjustment, which is entirely implementation defined.  
注意，此算法只能精确地估计给定的从机和主机之间的相位差。UAVCAN 没有定义时钟的速度/相位调整的算法，时钟的速度和相差是由具体实现定义的。

The following constants are defined for the time synchronization algorithm:  
时间同步算法定义如下常数：

 - T<sub>max</sub> - maximum broadcast interval for a given master.  
 - T<sub>max</sub> - 主机的最大广播间隔时间。
 <br><br/>
 - T<sub>min</sub> - minimum broadcast interval for a given master.  
 - T<sub>min</sub> - 主机的最小广播间隔时间。
 <br><br/>
 - T<sub>timeout</sub> - if the master was not broadcasting the time synchronization message for this amount of time, all slaves shall switch to the next active master with the highest priority.  
 - T<sub>timeout</sub> - 如果主服务端在这段时间内没有广播时间同步消息，所有从服务端都应该切换到下一个具有最高优先级的活跃主服务器。

The network may accommodate more than one time synchronization master working at the same time. In this case, only the master with the lowest node ID should be active; other masters should become passive by means of stopping broadcasting time synchronization messages, and they must synchronize with the active master instead. If the currently active master was not broadcasting time synchronization messages for the duration of Ttimeout, the next master with the highest priority becomes active instead, and all slaves will synchronize with it. When a higher priority master appears in the network, all other lower-priority masters should become passive, and all slaves will synchronize with the new master immediately.  
网络可以容纳多个时间同步主服务端同时工作。在这种情况下，只有具有最小节点 ID 的主节点是活动的；其他主服务端应该通过停止广播时间同步消息的方式变成被动的，它们必须与活跃的主服务端进行同步。如果当前活跃的主服务器在 T<sub>timeout</sub> 期间没有广播时间同步消息，那么具有最高优先级的下一个主服务器将变为活动的，所有从服务端将与它同步。当一个高优先级的主服务端出现在网络中时，所有其他低优先级的主服务端都应该变成被动的，所有从服务端都将立即与新主服务端同步。  

The message uavcan.protocol.GlobalTimeSync contains the exact timestamp of the previous transmission of this message. If the previous message was not yet transmitted, or if it was transmitted more than Tmax time units ago, the field must be set to zero.  
消息 uavcan.protocol.GlobalTimeSync 包含此消息的前一次传输的准确时间戳。如果之前尚未进行传输，或者传输的时间超过了 T<sub>max</sub> 时间单位，则必须将字段设置为 0。

It is recommended to refer to the existing UAVCAN implementations for reference.  
建议参考现有的 UAVCAN 实现。


#### Master （主端）
The following pseudocode describes the logic of a time synchronization master.  
下面的伪代码描述了时间同步主服务端的逻辑。

```
// State variables:
transfer_id := 0;
previous_tx_timestamp[NUM_IFACES];
previous_broadcast_timestamp;

// This function broadcasts a message with a specified Transfer ID using only one iface:
function broadcastMessage(transfer_id, iface_index, msg);

// This function returns the current value of a monotonic clock (the clock that doesn't change phase or rate)
function getMonotonicTime();

// This callback is invoked when the CAN driver completes transmission of a time sync message
// The tx_timestamp argument contains the exact timestamp when the CAN frame was delivered to the bus
function messageTxTimestampCallback(iface_index, tx_timestamp)
{
    previous_tx_timestamp[iface_index] := tx_timestamp;
}

// Publishes the message of type uavcan.protocol.GlobalTimeSync to each available interface
function broadcastTimeSync()
{
    current_time := getMonotonicTime();

    if (current_time - previous_broadcast_timestamp < MIN_PUBLICATION_PERIOD)
    {
        return;     // Rate limiting
    }

    if (current_time - previous_broadcast_timestamp > MAX_PUBLICATION_PERIOD)
    {
        for (i := 0; i < NUM_IFACES; i++)
        {
            previous_tx_timestamp[i] := 0;
        }
    }

    previous_broadcast_timestamp := current_time;

    message := uavcan.protocol.GlobalTimeSync();

    for (i := 0; i < NUM_IFACES; i++)
    {
        message.previous_transmission_timestamp_usec := previous_tx_timestamp[i];
        previous_tx_timestamp[i] := 0;
        broadcastMessage(transfer_id, i, message);
    }

    transfer_id++; // Overflow must be handled correctly
}
```


#### Slave (从端)
The following pseudocode describes the logic of a time synchronization slave.  
下面的伪代码描述了时间同步从端的逻辑。

```
// State variables:
previous_rx_real_timestamp := 0;               // This time is being synchronized
previous_rx_monotonic_timestamp := 0;     // This is the monotonic time (doesn't jump or change rate)
previous_transfer_id := 0;
state := STATE_UPDATE;       // STATE_UPDATE, STATE_ADJUST
master_node_id := -1;        // Invalid value
iface_index := -1;           // Invalid value

// This function performs local clock adjustment:
function adjustLocalTime(phase_error);

function adjust(message)
{
    // Clock adjustment will be performed every second message
    local_time_phase_error := previous_rx_real_timestamp - msg.previous_transmission_timestamp_usec;
    adjustLocalTime(local_time_phase_error);
    state := STATE_UPDATE;
}

function update(message)
{
    // Message is assumed to have two timestamps:
    //   Real - sampled from the clock that is being synchronized
    //   Monotonic - clock that never jumps and never changes rate
    previous_rx_real_timestamp := message.rx_real_timestamp;
    previous_rx_monotonic_timestamp := message.rx_monotonic_timestamp;
    master_node_id := message.source_node_id;
    iface_index := message.iface_index;
    previous_transfer_id := message.transfer_id;
    state := STATE_ADJUST;
}

// Accepts the message of type uavcan.protocol.GlobalTimeSync (please refer to the DSDL definition)
function handleReceivedTimeSyncMessage(message)
{
    time_since_previous_msg := message.monotonic_timestamp - previous_rx_monotonic_timestamp;

    // Resolving the state flags:
    needs_init := (master_node_id < 0) or (iface_index < 0);
    switch_master := message.source_node_id < master_node_id;
    publisher_timed_out := time_since_previous_msg > PUBLISHER_TIMEOUT;

    if (needs_init or switch_master or publisher_timed_out)
    {
        update(message);
    }
    else if ((message.iface_index == iface_index) and (message.source_node_id == master_node_id))
    {
        // Revert the state to STATE_UPDATE if needed
        if (state == STATE_ADJUST)
        {
            msg_invalid := message.previous_transmission_timestamp_usec == 0;
            wrong_tid := message.transfer_id != (previous_transfer_id + 1);    // Overflow must be handled correctly
            wrong_timing := time_since_previous_msg > MAX_PUBLICATION_PERIOD;
            if (msg_invalid or wrong_tid or wrong_timing)
            {
                state := STATE_UPDATE;
            }
        }
        // Handle the current state
        if (state == STATE_ADJUST)
        {
            adjust(message);
        }
        else
        {
            update(message);
        }
    }
    else
    {
        ; // Ignore this message
    }
}
```

__uavcan.protocol.GlobalTimeSync__

Default data type ID: 4
```
#
# Global time synchronization.
# Any node that publishes timestamped data must use this time reference.
#
# Please refer to the specification to learn about the synchronization algorithm.
#

#
# Broadcasting period must be within this range.
#
uint16 MAX_BROADCASTING_PERIOD_MS = 1100            # Milliseconds
uint16 MIN_BROADCASTING_PERIOD_MS = 40              # Milliseconds

#
# Synchronization slaves may switch to a new source if the current master was silent for this amount of time.
#
uint16 RECOMMENDED_BROADCASTER_TIMEOUT_MS = 2200    # Milliseconds

#
# Time in microseconds when the PREVIOUS GlobalTimeSync message was transmitted.
# If this message is the first one, this field must be zero.
#
truncated uint56 previous_transmission_timestamp_usec # Microseconds
```

### Node configuration (节点配置)
UAVCAN defines standard services for management of remote node’s configuration parameters. Support for these services is not mandatory but is highly recommended. The services are as follows:  
UAVCAN 定义了管理远程节点配置参数的标准服务。对这些服务的支持不是强制性的，但强烈建议这样做。服务内容如下：  

 - __uavcan.protocol.param.GetSet__ - gets or sets a single configuration parameter value, either by name or by index.  
 - __uavcan.protocol.param.GetSet__ - 获取或设置单个配置参数值，通过名称或索引获取。
 - __uavcan.protocol.param.ExecuteOpcode__ - allows control of the node configuration, including saving the configuration into the non-volatile memory, or resetting the configuration to default settings.  
  - __uavcan.protocol.param.ExecuteOpcode__ - 配置控制节点，包括将配置保存到非易失性内存中，或将配置重置为默认设置。
 - __uavcan.protocol.RestartNode__ - restarts a node remotely. Some nodes may require a restart before new configuration parameters can be applied.  
 - __uavcan.protocol.RestartNode__ - 远程重新启动节点。在应用新的配置参数之前，有些节点可能需要重新启动。

In some cases, a node may require more complex configuration than can be conveniently managed via these services. If this is the case, the recommendation is to manage the node’s configuration through configuration files accessible via the standard file management services (documented in this section).  
在某些情况下，节点可能需要比通过这些服务更复杂的配置。如果是这种情况，建议通过可通过标准文件管理服务访问的配置文件来管理节点的配置（方法见本节）。

__uavcan.protocol.param.ExecuteOpcode__
Default data type ID: 10
```
#
# Service to control the node configuration.
#

#
# SAVE operation instructs the remote node to save the current configuration parameters into a non-volatile
# storage. The node may require a restart in order for some changes to take effect.
#
# ERASE operation instructs the remote node to clear its configuration storage and reinitialize the parameters
# with their default values. The node may require a restart in order for some changes to take effect.
#
# Other opcodes may be added in the future (for example, an opcode for switching between multiple configurations).
#
uint8 OPCODE_SAVE  = 0  # Save all parameters to non-volatile storage.
uint8 OPCODE_ERASE = 1  # Clear the non-volatile storage; some changes may take effect only after reboot.
uint8 opcode

#
# Reserved, keep zero.
#
int48 argument

---

#
# If 'ok' (the field below) is true, this value is not used and must be kept zero.
# If 'ok' is false, this value may contain error code. Error code constants may be defined in the future.
#
int48 argument

#
# True if the operation has been performed successfully, false otherwise.
#
bool ok
```

__uavcan.protocol.param.GetSet__
Default data type ID: 11
```
#
# Get or set a parameter by name or by index.
# Note that access by index should only be used to retreive the list of parameters; it is higly
# discouraged to use it for anything else, because persistent ordering is not guaranteed.
#

#
# Index of the parameter starting from 0; ignored if name is nonempty.
# Use index only to retrieve the list of parameters.
# Parameter ordering must be well defined (e.g. alphabetical, or any other stable ordering),
# in order for the index access to work.
#
uint13 index

#
# If set - parameter will be assigned this value, then the new value will be returned.
# If not set - current parameter value will be returned.
# Refer to the definition of Value for details.
#
Value value

#
# Name of the parameter; always preferred over index if nonempty.
#
uint8[<=92] name

---

void5

#
# Actual parameter value.
#
# For set requests, it should contain the actual parameter value after the set request was
# executed. The objective is to let the client know if the value could not be updated, e.g.
# due to its range violation, etc.
#
# Empty value (and/or empty name) indicates that there is no such parameter.
#
Value value

void5
Value default_value    # Optional

void6
NumericValue max_value # Optional, not applicable for bool/string

void6
NumericValue min_value # Optional, not applicable for bool/string

#
# Empty name (and/or empty value) in response indicates that there is no such parameter.
#
uint8[<=92] name
```

__uavcan.protocol.param.Empty__
```
#
# Ex nihilo nihil fit.
#

```

__uavcan.protocol.param.NumericValue__
```
#
# Numeric-only value.
#
# This is a union, which means that this structure can contain either one of the fields below.
# The structure is prefixed with tag - a selector value that indicates which particular field is encoded.
#

union                          # Tag is 2 bits long.

Empty empty                     # Empty field, used to represent an undefined value.

int64   integer_value
float32 real_value

```

__uavcan.protocol.param.Value__
```
#
# Single parameter value.
#
# This is a union, which means that this structure can contain either one of the fields below.
# The structure is prefixed with tag - a selector value that indicates which particular field is encoded.
#

union                          # Tag is 3 bit long, so outer structure has 5-bit prefix to ensure proper alignment

Empty empty                     # Empty field, used to represent an undefined value.

int64        integer_value
float32      real_value         # 32-bit type is used to simplify implementation on low-end systems
uint8        boolean_value      # 8-bit value is used for alignment reasons
uint8[<=128] string_value       # Length prefix is exactly one byte long, which ensures proper alignment of payload
```

#### Standard configuration parameters (标准配置参数)
There are some configuration parameters that are common for most UAVCAN nodes. Examples of such common parameters include message publication frequencies, non-default data type ID settings, local node ID, etc. The UAVCAN specification improves compatibility by providing the following naming conventions for UAVCAN-related configuration parameters. Following these conventions is highly encouraged, but not mandatory.  
对于大多数 UAVCAN 节点都包含一些通用的配置参数。这些通用参数的示例包括消息发布频率、非默认数据类型 ID 设置、本地节点 ID 等。UAVCAN 规范通过提供以下与 UAVCAN 相关的配置参数的命名约定来改进兼容性。强烈建议遵循这些约定，但不是强制性的。

As can be seen below, all standard UAVCAN-related parameters share the same prefix uavcan..  
如下所示，所有标准的 UAVCAN 相关的参数，共享相同的前缀 uavcan. 。

##### Data type ID (数据类型 ID)
Parameter name: uavcan.dtid-X, where X stands for the full data type name, e.g. uavcan.dtid-uavcan.protocol.NodeStatus.  
参数名称：uavcan.dtid-X，其中 X 表示完整的数据类型名称，例如：uavcan.dtid-uavcan.protocol.NodeStatus。

This parameter configures the data type ID value for a given data type.  
此参数用于配置给定数据类型的数据类型 ID 值。

##### Message publication period （消息发布周期）
Parameter name: uavcan.pubp-X, where X stands for the full data type name; e.g. uavcan.pubp-uavcan.protocol.NodeStatus.  
参数名称：uavcan.pubp-X，其中 X 表示完整的数据类型名称，例如：uavcan.pubp-uavcan.protocol.NodeStatus。

This parameter configures the publication period for a given data type, in integer number of microseconds. Zero value means that publication should be disabled.  
此参数用于配置给定数据类型的发布周期（以微秒为单位的整数）。零值意味着发布应该被禁止。

##### Transfer priority （传输优先级）
Parameter name: uavcan.prio-X, where X stands for the full data type name, e.g. uavcan.prio-uavcan.protocol.NodeStatus.  
参数名称：uavcan.prio-X，其中 X 表示完整的数据类型名称，例如：uavcan.prio-uavcan.protocol.NodeStatus。

This parameter configures the transport priority level that will be used when publishing messages or calling services of a given data type.  
此参数用于配置在发布消息或调用给定数据类型的服务时的传输优先级。

##### Node ID （节点 ID）
Parameter name: uavcan.node_id.  
参数名称：uavcan.node_id。

This parameter configures ID of the local node. Zero means that the node ID is unconfigured, which may prompt the node to resort to dynamic node ID allocation after startup.  
此参数配置本地节点的 ID。0 表示节点 ID 未配置，用于提示节点在启动后进行动态节点 ID 分配。

##### CAN bus bit rate （CAN 总线的比特速率）
Parameter name: uavcan.bit_rate.  
参数名称：uavcan.bit_rate。

This parameter configures CAN bus bit rate. Zero value should trigger automatic bit rate detection, which should be the default option. Please refer to the hardware design recommendations for recommended values and other details.  
该参数可以配置 CAN 总线的比特率。零值会触发自动比特率检测，这是默认选项。有关推荐值和其他详细信息，请参阅硬件设计建议。

##### Instance ID （实例 ID）
Parameter name: uavcan.id-X-Y, where X is namespace name; Y is ID field name.  
参数名称：uavcan.id-X-Y。其中 X 为命名空间的名称；Y 是 ID 字段名。

Some UAVCAN messages (standard and possibly vendor-specific ones) use special fields that identify the instance of a certain function - ID fields. For example, messages related to actuator control use fields named actuator_id, some sensor messages use fields named sensor_id, etc. In order to improve compatibility, the specification offers a naming convention for parameters that define the values used in ID fields.  
部分 UAVCAN 消息（标准的和特定于供应商的消息）使用特殊的字段来标识某个函数的实例—— ID 字段。例如，与执行器控制相关的消息使用名为actuator_id的字段，一些传感器消息使用名为sensor_id的字段，等等。为了提高兼容性，该规范为定义 ID 字段中使用的值的参数提供了命名约定。

Given messages located in the namespace X that share an ID field named Y, the corresponding parameter name would be uavcan.id-X-Y. For example, the parameter for the field esc_index that is used in the message uavcan.equipment.esc.Status and that defines the array index in uavcan.equipment.esc.RawCommand, will be named as follows:  
位于命名空间 X 中共享 ID 字段为 Y 的消息，相应的参数名称将是 uavcan.id-X-Y。例如，消息 uavcan.equipment.esc.Status 中使用的字段 esc_index 的参数，它定义 uavcan.equipment.esc.RawCommand 中的数组索引，其名称如下:

__uavcan.id-uavcan.equipment.esc-esc_index__

In the case that an ID field is shared across different namespaces, then the most common outer shared namespace should be used as X. This is not the case for any of the standard messages, so an example cannot be provided.  
如果一个 ID 字段在不同的命名空间之间共享，那么最常见的外部共享命名空间应该被用作 x。这不是一种标准消息的情况，因此无法提供示例。


In the case that an ID field is used in the standard namespace (uavcan.*) and in some vendor-specific namespaces at the same time, the prefix should be used as though the ID field was used only in the standard namespace.  
如果在标准命名空间（uavcan.*）中使用了 ID 字段，同时在一些特定于供应商的名称空间中使用了ID 字段，那么应该使用前缀，就像只在标准名称空间中使用 ID 字段一样。


### File transfer （文件传输）
File transfer is a very generic feature of UAVCAN, that allows access to the file system on remote nodes. The feature is based upon a set of UAVCAN services that are listed below.  
文件传输是 UAVCAN 的一个常规功能，它用于访问远程节点上的文件系统。该功能基于下面列出的一组 UAVCAN 服务。

#### Firmware update (固件更新)
In terms of UAVCAN, firmware update is a special case of file transfer. The process of firmware update involves two or three nodes:  
对于 UAVCAN 来说，固件更新是文件传输的一个特例。固件更新过程涉及 2 - 3 个节点：

 - The node that initiates the process of firmware update, or updater.  
 - 发起固件更新或更新程序的节点。
<br><br/>
 - The node that provides access to the firmware file, or file server. In most cases, the updater will be acting as a file server and only two nodes are involved.  
 - 提供对固件文件或文件服务端的访问的节点。在大多数情况下，更新端将充当文件服务端，因此只涉及两个节点。
<br><br/>
 - The node that is being updated, or updatee.  
 - 正在进行更新或即将被更新的节点。

The process can be described as follows:  
更新的处理过程如下：

 - The updater decides that a certain node (the updatee) should be updated.  
 - 更新端决定哪个节点（被更新端）应该被更新。
<br><br/>
 - The updater invokes the service uavcan.protocol.file.BeginFirmwareUpdate on the updatee. The information about the location of the firmware file will be passed to the updatee via the service request.  
 - 发起更新端通过调用 uavcan.protocol.file.BeginFirmwareUpdate 服务在被更新端上发起更新。相关的本地的固件信息会通过服务请求传输到被更新端。
<br><br/>
 - If the updatee chooses to accept the update request, it performs initialization procedures as required by its implementation (e.g., rebooting into the bootloader, etc).  
 - 如果被更新端选择接受更新请求，它将执行必须的初始化过程（例如，重新启动到引导加载程序，等等）。
 <br><br/>
 - The updatee receives new firmware file from the file server using information received via the service request above.  
 - 被更新者通过上述服务请求接收的信息从文件服务器接收新固件文件，并用其更新。
 <br><br/>
 - The updatee completes the update and restarts.  
 - 被更新端完成更新并重新启动。


Typically, the updatee will also resort to the dynamic node ID allocation process, which is documented in this section.  
通常，被更新端还将使用动态节点 ID 分配过程，本节对此进行了说明。

__uavcan.protocol.file.BeginFirmwareUpdate__
Default data type ID: 40
```
#
# This service initiates firmware update on a remote node.
#
# The node that is being updated (slave) will retrieve the firmware image file 'image_file_remote_path' from the node
# 'source_node_id' using the file read service, then it will update the firmware and reboot.
#
# The slave can explicitly reject this request if it is not possible to update the firmware at the moment
# (e.g. if the node is busy).
#
# If the slave node accepts this request, the initiator will get a response immediately, before the update process
# actually begins.
#
# While the firmware is being updated, the slave should set its mode (uavcan.protocol.NodeStatus.mode) to
# MODE_SOFTWARE_UPDATE.
#

uint8 source_node_id         # If this field is zero, the caller's Node ID will be used instead.
Path image_file_remote_path

---

#
# Other error codes may be added in the future.
#
uint8 ERROR_OK               = 0
uint8 ERROR_INVALID_MODE     = 1    # Cannot perform the update in the current operating mode or state.
uint8 ERROR_IN_PROGRESS      = 2    # Firmware update is already in progess, and the slave doesn't want to restart.
uint8 ERROR_UNKNOWN          = 255
uint8 error

uint8[<128] optional_error_message   # Detailed description of the error.

```

__uavcan.protocol.file.GetInfo__
Default data type ID: 45

```
#
# Request info about a remote file system entry (file, directory, etc).
#

Path path

---

#
# File size in bytes.
# Should be set to zero for directories.
#
uint40 size

Error error

EntryType entry_type
```

__uavcan.protocol.file.GetDirectoryEntryInfo__
Default data type ID: 46

```
#
# This service can be used to retrieve remote directory listing, one entry per request.
#
# The client should query each entry independently, iterating 'entry_index' from 0 until the last entry is passed,
# in which case the server will report that there is no such entry (via the fields 'entry_type' and 'error').
#
# The entry_index shall be applied to the ordered list of directory entries (e.g. alphabetically ordered). The exact
# sorting criteria does not matter as long as it provides the same ordering for subsequent service calls.
#

uint32 entry_index

Path directory_path

---

Error error

EntryType entry_type

Path entry_full_path  # Ignored/Empty if such entry does not exist.

```

__uavcan.protocol.file.Delete__
Default data type ID: 47

```
#
# Delete remote file system entry.
# If the remote entry is a directory, all nested entries will be removed too.
#

Path path

---

Error error
```

__uavcan.protocol.file.Read__
Default data type ID: 48

```
#
# Read file from a remote node.
# 
# There are two possible outcomes of a successful service call:
#  1. Data array size equals its capacity. This means that the end of the file is not reached yet.
#  2. Data array size is less than its capacity, possibly zero. This means that the end of file is reached.
# 
# Thus, if the client needs to fetch the entire file, it should repeatedly call this service while increasing the
# offset, until incomplete data is returned.
#
# If the object pointed by 'path' cannot be read (e.g. it is a directory or it does not exist), appropriate error code
# will be returned, and data array will be empty.
#

uint40 offset

Path path

---

Error error

uint8[<=256] data

```

__uavcan.protocol.file.Write__
Default data type ID: 49

```
#
# Write into a remote file.
# The server shall place the contents of the field 'data' into the file pointed by 'path' at the offset specified by
# the field 'offset'.
#
# When writing a file, the client should repeatedly call this service with data while advancing offset until the file
# is written completely. When write is complete, the client shall call the service one last time, with the offset
# set to the size of the file and with the data field empty, which will signal the server that the write operation is
# complete.
#
# When the write operation is complete, the server shall truncate the resulting file past the specified offset.
#
# Server implementation advice:
# It is recommended to implement proper handling of concurrent writes to the same file from different clients, for
# example by means of creating a staging area for uncompleted writes (like FTP servers do).
#

uint40 offset

Path path

uint8[<=192] data

---

Error error

```

__uavcan.protocol.file.EntryType__
```
#
# Nested type.
# Represents the type of the file system entry (e.g. file or directory).
# If such entry does not exist, 'flags' must be set to zero.
#

uint8 FLAG_FILE      = 1        # Excludes FLAG_DIRECTORY
uint8 FLAG_DIRECTORY = 2        # Excludes FLAG_FILE
uint8 FLAG_SYMLINK   = 4        # Link target is either FLAG_FILE or FLAG_DIRECTORY
uint8 FLAG_READABLE  = 8
uint8 FLAG_WRITEABLE = 16

uint8 flags

```


__uavcan.protocol.file.Error__
```
#
# Nested type.
# File operation result code.
#

int16 OK                = 0
int16 UNKNOWN_ERROR     = 32767

int16 NOT_FOUND         = 2
int16 IO_ERROR          = 5
int16 ACCESS_DENIED     = 13
int16 IS_DIRECTORY      = 21 # I.e. attempt to read/write on a path that points to a directory
int16 INVALID_VALUE     = 22 # E.g. file name is not valid for the target file system
int16 FILE_TOO_LARGE    = 27
int16 OUT_OF_SPACE      = 28
int16 NOT_IMPLEMENTED   = 38

int16 value
```

__uavcan.protocol.file.Path__
```
#
# Nested type.
#
# File system path in UTF8.
#
# The only valid separator is forward slash.
#

uint8 SEPARATOR = '/'

uint8[<=200] path

```

### Debug features (调试功能)
The following messages are designed to facilitate debugging and to provide means of reporting events in a human-readable representation.
以下消息的设计目的是为了方便调试，并提供以可读的表示形式报告事件的方法。

__uavcan.protocol.debug.KeyValue__
Default data type ID: 16370
```
#
# Generic named parameter (key/value pair).
#

#
# Integers are exactly representable in the range (-2^24, 2^24) which is (-16'777'216, 16'777'216).
#
float32 value

#
# Tail array optimization is enabled, so if key length does not exceed 3 characters, the whole
# message can fit into one CAN frame. The message always fits into one CAN FD frame.
#
uint8[<=58] key

```

__uavcan.protocol.debug.LogMessage__
Default data type ID: 16383
```
#
# Generic log message.
# All items are byte aligned.
#

LogLevel level
uint8[<=31] source
uint8[<=90] text

```

__uavcan.protocol.debug.LogLevel__
```
#
# Log message severity
#

uint3 DEBUG    = 0
uint3 INFO     = 1
uint3 WARNING  = 2
uint3 ERROR    = 3
uint3 value

```

### Command shell access （命令shell访问）
The following service allows execution of arbitrary commands on a remote node via direct access to its internal command shell.  
以下服务允许直接通过shell命令访问远程节点。

__uavcan.protocol.AccessCommandShell__
Default data type ID: 6
```
#
# THIS DEFINITION IS SUBJECT TO CHANGE.
#
# This service allows to execute arbitrary commands on the remote node's internal system shell.
#
# Essentially, this service mimics a typical terminal emulator, with one text input (stdin) and two text
# outputs (stdout and stderr). When there's no process running, the input is directed into the terminal
# handler itself, which interpretes it. If there's a process running, the input will be directed into
# stdin of the running process. It is possible to forcefully return the terminal into a known state by
# means of setting the reset flag (see below), in which case the terminal will kill all of the child
# processes, if any, and return into the initial idle state.
#
# The server is assumed to allocate one independent terminal instance per client, so that different clients
# can execute commands without interfering with each other.
#

#
# Input and output should use this newline character.
#
uint8 NEWLINE = '\n'

#
# The server is required to keep the result of the last executed command for at least this time.
# When this time expires, the server may remove the results in order to reclaim the memory, but it
# is not guaranteed. Hence, the clients must retrieve the results in this amount of time.
#
uint8 MIN_OUTPUT_LIFETIME_SEC = 10

#
# These flags control the shell and command execution.
#
uint8 FLAG_RESET_SHELL          = 1     # Restarts the shell instance anew; may or may not imply CLEAR_OUTPUT_BUFFERS
uint8 FLAG_CLEAR_OUTPUT_BUFFERS = 2     # Makes stdout and stderr buffers empty
uint8 FLAG_READ_STDOUT          = 64    # Output will contain stdout
uint8 FLAG_READ_STDERR          = 128   # Output will be extended with stderr
uint8 flags

#
# If the shell is idle, it will interpret this string.
# If there's a process running, this string will be piped into its stdin.
#
# If RESET_SHELL is set, new input will be interpreted by the shell immediately.
#
uint8[<=128] input

---

#
# Exit status of the last executed process, or error code of the shell itself.
# Default value is zero.
#
int32 last_exit_status

#
# These flags indicate the status of the shell.
#
uint8 FLAG_RUNNING              = 1     # The shell is currently running a process; stdin/out/err are piped to it
uint8 FLAG_SHELL_ERROR          = 2     # Exit status contains error code, output contains text (e.g. no such command)
uint8 FLAG_HAS_PENDING_STDOUT   = 64    # There is more stdout to read
uint8 FLAG_HAS_PENDING_STDERR   = 128   # There is more stderr to read
uint8 flags

#
# In case of a shell error, this string may contain ASCII string explaining the nature of the error.
# Otherwise, if stdout read is requested, this string will contain stdout data. If stderr read is requested,
# this string will contain stderr data. If both stdout and stderr read is requested, this string will start
# with stdout and end with stderr, with no separator in between.
#
uint8[<=256] output

```

### Panic mode (应急模式)
The panic message allows the broadcaster to quickly shut down the system in the event of an emergency.  
应急信息允许通过发布广播消息在紧急情况下迅速关闭系统。

__uavcan.protocol.Panic__
Default data type ID: 5
```
#
# This message may be published periodically to inform network participants that the system has encountered
# an unrecoverable fault and is not capable of further operation.
#
# Nodes that are expected to react to this message should wait for at least MIN_MESSAGES subsequent messages
# with any reason text from any sender published with the interval no higher than MAX_INTERVAL_MS before
# undertaking any emergency actions.
#

uint8 MIN_MESSAGES = 3

uint16 MAX_INTERVAL_MS = 500

#
# Short description that would fit a single CAN frame.
#
uint8[<=7] reason_text

```

### Dynamic node ID allocation (动态分配节点 ID)
In order to be able to operate in a UAVCAN network, a node must have a node ID that is unique within the network. Typically, a valid node ID can be configured manually for each node; however, in certain use cases the manual approach is either undesirable or impossible, therefore UAVCAN defines the high-level feature of dynamic node ID allocation, that allows nodes to obtain a node ID value automatically upon connection to the network.  
为了能够在 UAVCAN 网络中进行工作，每个节点必须具有在网络中惟一的节点 ID。通常，可以为每个节点手动配置有效的节点 ID；然而，在某些用例中，手动方式不受欢迎或者不可能实现，因此 UAVCAN 定义了动态节点 ID 分配的高级特性，允许节点在连接到网络时自动获得节点 ID 值。

Dynamic node ID allocation combined with automatic CAN bus bit rate detection makes it easy to implement nodes that can join any UAVCAN network without any manual configuration. These sorts of nodes are referred to as plug-and-play nodes.  
动态节点 ID 分配与自动 CAN 总线比特率检测相结合，使得无需任何手动配置即可加入任何 UAVCAN 网络的节点易于实现。这类节点称为即插即用节点。

A dynamically allocated node ID cannot be persistent. This means that if a node is configured to use a dynamic node ID, it must perform a new allocation every time it starts or reboots.  
动态分配的节点 ID 不能被保持。这意味着，如果一个节点被配置为使用动态节点 ID，那么它必须在每次启动或重新引导时执行一个新的分配。

The process of dynamic node ID allocation always involves two types of nodes: allocators, which serve allocation requests; and allocatees, which request dynamic node ID from allocators. A UAVCAN network may implement the following configurations of allocators:  
动态节点 ID 分配过程通常涉及两类节点：分配器，提供分配服务；被分配器，从分配器请求动态节点 ID。UAVCAN 网络可以实现一下几种分配器设置：

 - Zero allocators, in which case the feature of dynamic node ID allocation will not be available.  
 - 0 个分配器，这种情况动态节点 ID 分配功能不可用。
 - One allocator, in which case the feature of dynamic node ID allocation will become unavailable if the allocator fails. In this configuration, the role of the allocator can be performed even by a very resource-constrained system, e.g. a low-end microcontroller.  
 - 仅 1 个分配器，在这种情况下，如果分配器故障，动态节点 ID 分配的特性将不可用。在这种配置中，分配器的角色甚至可以由资源非常有限的系统来执行，例如低端微控制器。
 - Three allocators, in which case the allocators will be using a replicated state via a distributed consensus algorithm. In this configuration, the network can tolerate the loss of one allocator and continue to serve allocation requests. This configuration requires that the allocators to maintain large data structures for the purposes of the distributed consensus algorithm, and may therefore require a slightly more sophisticated computational platform, e.g. a high-end microcontroller.  
 - 三个分配器，在这种情况下，分配器将通过分布式算法使用相互同步状态。在这种配置中，网络可以允许一个分配器的丢失，并继续为分配请求提供服务。这种配置要求分配器为分布式同步算法维护一个大型数据结构，因此可能需要稍微复杂一点的计算平台，例如高端微控制器。
 - Five allocators, is the same as the three allocator configuration except that the network can tolerate the loss of two allocators and still continue to serve allocation requests.  
 - 五个分配器，与三个分配器配置相同，不同之处在于网络可以容忍两个分配器的丢失，并且仍然可以继续为分配请求服务。


In order to get a dynamic node ID, each allocatee must have a globally unique 128-bit integer identifier, known as unique ID. This is the same value that is used in the field unique_id of the data type uavcan.protocol.HardwareVersion. Every node that requires a dynamic ID allocation must support the service uavcan.protocol.GetNodeInfo, and the nodes must use the same unique ID value during dynamic node ID allocation and when responding to uavcan.protocol.GetNodeInfo requests.  
为了获得动态节点 ID，每个申请分配者必须有一个全局惟一的 128 位整数标识符，称为唯一 ID。这个值与数据类型 uavcan.protocol.HardwareVersion 的 unique_id 字段中使用的值相同。每个需要动态 ID 分配的节点必须支持服务 uavcan.protocol.GetNodeInfo，节点必须使用同样的唯一 ID 值以响应动态节点 ID 申请的服务。

During dynamic allocation, the allocatee communicates its unique ID to the allocator (or allocators), which then use it to produce an appropriate allocation response. Unique ID values are kept by allocators in allocation tables - data structures that contain mappings between unique ID and corresponding node ID values. Allocation tables are write-only data structures that can only grow. Once a new allocatee has requested a node ID, its unique ID will be recorded into the allocation table, and all subsequent allocation requests from the same allocatee will be served with the same node ID value.  
在动态分配期间，被分配器将其惟一 ID 传递给一个或多个分配器，后者随后使用它来生成适当的分配响应。惟一 ID 值由分配表中的分配程序保存——数据结构包含惟一 ID 和对应节点 ID 值之间的映射。分配表是只写的数据结构，只能增长。一旦一个新的被分配器请求了一个节点 ID，它的唯一 ID 将被记录到分配表中，来自相同被分配器的所有后续分配请求都将使用相同的节点 ID 值。

In configurations with redundant allocators, every allocator maintains a replica of the same allocation table (a UAVCAN network cannot contain more than one allocation table, regardless of the number of allocators employed). While the allocation table is write-only data structure that can only grow, it is still possible to wipe the table completely, forcing the allocators to forget known nodes and perform all following allocations anew.  
在使用冗余分配器的配置中，每个分配器都维护同一个分配表的一个副本（一个 UAVCAN 网络只能使用同一个分配表，不管使用了多少个分配器）。虽然分配表是只能增长的写数据结构，但仍然有可能完全擦除该表，从而迫使分配器忘记已知节点，重新执行所有后续分配。

In the context of this chapter, nodes that are using dynamic node ID will be referred to as dynamic nodes, and nodes that are using manually-configured node ID will be referred to as static nodes. It is assumed that in most cases, allocators will be static nodes themselves (since there’s no other authority on the network that can grant dynamic node ID, allocators will not be able to dynamically allocate themselves). Excepting allocators, it is not recommended to mix dynamic and static nodes on the same network; i.e., normally, a UAVCAN network should contain either all static nodes, or all dynamic nodes (except allocators). In case if this recommendation cannot be followed, the following rules of safe co-existence of dynamic nodes with static nodes must be considered:  
在本章的上下文中，使用动态节点 ID 的节点将被称为动态节点，而使用手动配置的节点 ID 的节点将被称为静态节点。假设在大多数情况下，分配器本身是静态节点（因为网络上没有其他权威机构可以授予动态节点 ID，所以分配器不能动态地分配它们自己）。除分配器外，不建议在同一网络上混合使用动态和静态节点；通常，UAVCAN 网络允许包含所有静态节点或所有动态节点（分配器除外）。如果不能遵循此建议，则必须考虑以下动态节点与静态节点安全共存的规则:

 - It is safe to connect dynamic nodes to the bus at any time.  
 - 保证动态节点在任何时刻接入总线总是安全的。
<br><br/>
 - A static node can be connected to the bus if the allocator (allocators) is (are) already aware of them, i.e. these static nodes are already in the allocation table.  
- 如果分配器中已经包括一个静态节点，保证他可以正确被接入总线。
<br><br/>
 - A new static node (i.e. a node that does not meet the above condition) can be connected to the bus only if:  
 - 新静态节点（比如不符合上述条件的节点）只有在下列情况下才可连接到总线：
	<br><br/>
	- New dynamic allocations are not happening at the moment.  
	- 总线当前没有动态分配行为。
	<br><br/>
	- The allocators are capable of serving new allocations.  
	- 分配器能够重新提供新的分配。

As can be inferred from the above, the process of dynamic node ID allocation involves up to two types of communications:  
从上面可以推断，动态节点 ID 分配的过程涉及到两种类型的通信：

 - Allocatee-allocator - this communication is used when an allocatee requests a dynamic node ID from the allocator (allocators), and when the allocator (allocators) transmits a response back to the allocatee. This communication is invariant to the allocator configuration used, i.e., the allocatees are not aware of how many allocators are available on the network and how they are configured.  
 - 被分配器发送请求给分配器 —— 这种通信行为是指被分配器向分配器发送动态ID请求，而且分配器发送一个响应给被分配器的过程。这种通信过程不会随着分配器不同的配置而变化，即被分配器并不知道有多少个分配器存在于网络之中。
 
 - Allocator-allocator - this communication is used by allocators for the purpose of maintenance of the replicated allocation table and for other needs of the distributed consensus algorithm. Allocatees are completely isolated and unaware of these exchanges. This communication is not applicable for the single-allocator configuration.  
 - 分配器发送请求给其他分配器 —— 分配器使用此通信的目的是维护复制的分配表，并满足分布式一致算法的其他需求。这类通信与被分配器无关。此通信无法应用于单分配器配置。

#### Allocatee-allocator exchanges (分配器和被分配器的交换)
Allocatee-allocator exchanges are performed using only one message type - uavcan.protocol.dynamic_node_id.Allocation. Allocators use it with regular message broadcast transfers; allocatees use it with anonymous message transfers. The specification and usage info for this data type is provided below.  
分配器和被分配器的交换只使用一个消息类型 uavcan.protocol.dynamic_node_id.Allocation。分配器把他当作常规消息广播传输使用；被分配器把他当作匿名消息传输使用。下面提供了该数据类型的规范和使用信息。

The general idea of the allocatee-allocator exchanges is that the allocatee communicates to the allocator its unique ID and, if applicable, the preferred node ID value, using anonymous message transfers of type uavcan.protocol.dynamic_node_id.Allocation. The allocator performs the allocation and sends a response using the same message type, where the field for unique ID is populated with the unique ID of the requesting node and the field for node ID is populated with the allocated node ID. Note that since the allocator that serves the allocation always has a node ID, it is free to use multi-frame transfers, therefore the allocator can directly send the response using a single message transfer. The allocatees, however, are restricted to single-frame transfers, due to limitations of anonymous message transfers. Therefore, the allocatees send their unique ID to the allocator using three single-frame transfers, where the first transfer contains the first part of their unique ID, second transfer contains the continuation, and the last transfer contains the last few bytes of the unique ID. The details are provided in the DSDL description of the message type.  
分配器和被分配器互通的核心就是被分配器通过匿名消息传输发送他的唯一 ID 给分配器，发送时通过 uavcan.protocol.dynamic_node_id.Allocation 传输。分配器执行分配并使用相同的消息类型发送响应，其中唯一 ID 字段用收到请求的节点唯一 ID 填充，而节点 ID 字段由计算分配的节点 ID 填充。注意，由于服务于分配的分配器总是有一个节点ID，所以分配请求可以使用多帧传输，也可以使用单帧传输。然而，由于匿名消息传输的限制，被分配器只能使用单帧传输。因此，分配给他们的唯一 ID 分配器使用三个独立的帧发送，其中第一传输包含他们的唯一 ID 的第一部分，第二传输包含延续，最后一个传输包含唯一 ID 的最后几个字节。细节在 DSDL 消息类型的描述中提供。

The specification of the data type contains a description of the exchange protocol on the side of allocatee. On the allocator’s side the algorithm should be implemented as shown in the following pseudocode.  
数据类型的规范在被分配器方面有个对交换协议的说明。在分配器方面，算法应该如下面的伪代码所示来实现。

Please note that the pseudocode refers to a function named canPublishFollowupAllocationResponse(), which is only applicable in the case of a redundant allocator configuration. It evaluates the current state of the distributed consensus and decides whether the current node is allowed to engage in allocation exchanges. The logic of this function will be reviewed in the chapter dedicated to redundant allocators. In the non-redundant allocator configuration, this function will always return true, meaning that the allocator is always allowed to engage in allocation exchanges.  
请注意，伪代码引用了一个名为 canPublishFollowupAllocationResponse() 的函数，该函数仅适用于冗余分配器配置的情况。它评估分布式一致算法的当前状态，并决定当前节点是否允许当前分配器参与分配交换。这个函数的逻辑将在专门讨论冗余分配器的章节中进行讨论。在非冗余分配器配置中，此函数将始终返回 true，这意味着始终允许分配器参与分配交换。

```
// Constants:
InvalidStage = 0;

// State variables:
last_message_timestamp;
current_unique_id;

// This function will be invoked when a complete unique ID is received.
// Typically, the actual allocation will be carried out in this function.
function handleAllocationRequest(unique_id, preferred_node_id);

// This function is only applicable in a configuration with redundant allocators.
// Its return value depends on the current state of the distributed consensus algorithm.
// Please refer to the allocator-allocator communication logic for details.
// In the non-redundant configuration this function will always return true.
function canPublishFollowupAllocationResponse();

// This is an internal function; see below.
function detectRequestStage(msg)
{
    if ((msg.unique_id.size() != MAX_LENGTH_OF_UNIQUE_ID_IN_REQUEST) &&
        (msg.unique_id.size() != (msg.unique_id.capacity() - MAX_LENGTH_OF_UNIQUE_ID_IN_REQUEST * 2U)) &&
        (msg.unique_id.size() != msg.unique_id.capacity()))     // For CAN FD
    {
        return InvalidStage;
    }
    if (msg.first_part_of_unique_id)
    {
        return 1;       // Note that CAN FD frames can deliver the unique ID in one stage!
    }
    if (msg.unique_id.size() == MAX_LENGTH_OF_UNIQUE_ID_IN_REQUEST)
    {
        return 2;
    }
    if (msg.unique_id.size() < MAX_LENGTH_OF_UNIQUE_ID_IN_REQUEST)
    {
        return 3;
    }
    return InvalidStage;
}

// This is an internal function; see below.
function getExpectedStage()
{
    if (current_unique_id.empty())
    {
        return 1;
    }
    if (current_unique_id.size() >= (MAX_LENGTH_OF_UNIQUE_ID_IN_REQUEST * 2))
    {
        return 3;
    }
    if (current_unique_id.size() >= MAX_LENGTH_OF_UNIQUE_ID_IN_REQUEST)
    {
        return 2;
    }
    return InvalidStage;
}

// This function is invoked when the allocator receives a message of type uavcan.protocol.dynamic_node_id.Allocation.
function handleAllocation(msg)
{
    if (!msg.isAnonymousTransfer())
    {
        return;         // This is a response from another allocator, ignore
    }

    // Reset the expected stage on timeout
    if (msg.getMonotonicTimestamp() > (last_message_timestamp + FOLLOWUP_TIMEOUT))
    {
        current_unique_id.clear();
    }

    // Checking if request stage matches the expected stage
    request_stage = detectRequestStage(msg);
    if (request_stage == InvalidStage)
    {
        return;             // Malformed request - ignore without resetting
    }

    if (request_stage != getExpectedStage())
    {
        return;             // Ignore - stage mismatch
    }

    if (msg.unique_id.size() > current_unique_id.capacity() - current_unique_id.size())
    {
        return;             // Malformed request
    }

    // Updating the local state
    for (i = 0; i < msg.unique_id.size(); i++)
    {
        current_unique_id.push_back(msg.unique_id[i]);
    }

    if (current_unique_id.size() == current_unique_id.capacity())
    {
        // Proceeding with allocation.
        handleAllocationRequest(current_unique_id, msg.node_id);
        current_unique_id.clear();
    }
    else
    {
        // Publishing the follow-up if possible.
        if (canPublishFollowupAllocationResponse())
        {
            msg = uavcan.protocol.dynamic_node_id.Allocation();
            msg.unique_id = current_unique_id;
            broadcast(msg);
        }
        else
        {
            current_unique_id.clear();
        }
    }

    // It is important to update the timestamp only if the request has been processed successfully.
    last_message_timestamp = msg.getMonotonicTimestamp();
}

```

__uavcan.protocol.dynamic_node_id.Allocation__
Default data type ID: 1
```
#
# This message is used for dynamic Node ID allocation.
#
# When a node needs to request a node ID dynamically, it will transmit an anonymous message transfer of this type.
# In order to reduce probability of CAN ID collisions when multiple nodes are publishing this request, the CAN ID
# field of anonymous message transfer includes a Discriminator, which is a special field that has to be filled with
# random data by the transmitting node. Since Discriminator collisions are likely to happen (probability approx.
# 0.006%), nodes that are requesting dynamic allocations need to be able to handle them correctly. Hence, a collision
# resolution protocol is defined (alike CSMA/CD). The collision resolution protocol is based on two randomized
# transmission intervals:
#
# - Request period - Trequest.
# - Followup delay - Tfollowup.
#
# Recommended randomization ranges for these intervals are documented in the costants of this message type (see below).
# Random intervals must be chosen anew per transmission, whereas the Discriminator value is allowed to stay constant
# per node.
#
# In the below description the following terms are used:
# - Allocator - the node that serves allocation requests.
# - Allocatee - the node that requests an allocation from the Allocator.
#
# The response timeout is not explicitly defined for this protocol, as the Allocatee will request the allocation
# Trequest units of time later again, unless the allocation has been granted. Despite this, the implementation can
# consider the value of FOLLOWUP_TIMEOUT_MS as an allocation timeout, if necessary.
#
# On the allocatee's side the protocol is defined through the following set of rules:
#
# Rule A. On initialization:
# 1. The allocatee subscribes to this message.
# 2. The allocatee starts the Request Timer with a random interval of Trequest.
#
# Rule B. On expiration of Request Timer:
# 1. Request Timer restarts with a random interval of Trequest.
# 2. The allocatee broadcasts a first-stage Allocation request message, where the fields are assigned following values:
#    node_id                 - preferred node ID, or zero if the allocatee doesn't have any preference
#    first_part_of_unique_id - true
#    unique_id               - first MAX_LENGTH_OF_UNIQUE_ID_IN_REQUEST bytes of unique ID
#
# Rule C. On any Allocation message, even if other rules also match:
# 1. Request Timer restarts with a random interval of Trequest.
#
# Rule D. On an Allocation message WHERE (source node ID is non-anonymous) AND (allocatee's unique ID starts with the
# bytes available in the field unique_id) AND (unique_id is less than 16 bytes long):
# 1. The allocatee waits for Tfollowup units of time, while listening for other Allocation messages. If an Allocation
#    message is received during this time, the execution of this rule will be terminated. Also see rule C.
# 2. The allocatee broadcasts a second-stage Allocation request message, where the fields are assigned following values:
#    node_id                 - same value as in the first-stage
#    first_part_of_unique_id - false
#    unique_id               - at most MAX_LENGTH_OF_UNIQUE_ID_IN_REQUEST bytes of local unique ID with an offset
#                              equal to number of bytes in the received unique ID
#
# Rule E. On an Allocation message WHERE (source node ID is non-anonymous) AND (unique_id fully matches allocatee's
# unique ID) AND (node_id in the received message is not zero):
# 1. Request Timer stops.
# 2. The allocatee initializes its node_id with the received value.
# 3. The allocatee terminates subscription to Allocation messages.
# 4. Exit.
#

#
# Recommended randomization range for request period.
#
# These definitions have an advisory status; it is OK to pick higher values for both bounds, as it won't affect
# protocol compatibility. In fact, it is advised to pick higher values if the target application is not concerned
# about the time it will spend on completing the dynamic node ID allocation procedure, as it will reduce
# interference with other nodes, possibly of higher importance.
#
# The lower bound shall not be lower than FOLLOWUP_TIMEOUT_MS, otherwise the request may conflict with a followup.
#
uint16 MAX_REQUEST_PERIOD_MS = 1000     # It is OK to exceed this value
uint16 MIN_REQUEST_PERIOD_MS = 600      # It is OK to exceed this value

#
# Recommended randomization range for followup delay.
# The upper bound shall not exceed FOLLOWUP_TIMEOUT_MS, because the allocator will reset the state on its end.
#
uint16 MAX_FOLLOWUP_DELAY_MS = 400
uint16 MIN_FOLLOWUP_DELAY_MS = 0        # Defined only for regularity; will always be zero.

#
# Allocator will reset its state if there was no follow-up request in this amount of time.
#
uint16 FOLLOWUP_TIMEOUT_MS = 500

#
# Any request message can accommodate no more than this number of bytes of unique ID.
# This limitation is needed to ensure that all request transfers are single-frame.
# This limitation does not apply to CAN FD transport.
#
uint8 MAX_LENGTH_OF_UNIQUE_ID_IN_REQUEST = 6

#
# When requesting an allocation, set the field 'node_id' to this value if there's no preference.
#
uint7 ANY_NODE_ID = 0

#
# If transfer is anonymous, this is the preferred ID.
# If transfer is non-anonymous, this is allocated ID.
#
# If the allocatee does not have any preference, this value must be set to zero. In this case, the allocator
# must choose the highest unused node ID value for this allocation (except 126 and 127, that are reserved for
# network maintenance tools). E.g., if the allocation table is empty and the node has requested an allocation
# without any preference, the allocator will grant the node ID 125.
#
# If the preferred node ID is not zero, the allocator will traverse the allocation table starting from the
# prefferred node ID upward, untill a free node ID is found. If a free node ID could not be found, the
# allocator will restart the search from the preferred node ID downward, until a free node ID is found.
#
# In pseudocode:
#   int findFreeNodeID(const int preferred)
#   {
#       // Search up
#       int candidate = (preferred > 0) ? preferred : 125;
#       while (candidate <= 125)
#       {
#           if (!isOccupied(candidate))
#               return candidate;
#           candidate++;
#       }
#       // Search down
#       candidate = (preferred > 0) ? preferred : 125;
#       while (candidate > 0)
#       {
#           if (!isOccupied(candidate))
#               return candidate;
#           candidate--;
#       }
#       // Not found
#       return -1;
#   }
#
uint7 node_id

#
# If transfer is anonymous, this field indicates first-stage request.
# If transfer is non-anonymous, this field should be assigned zero and ignored.
#
bool first_part_of_unique_id

#
# If transfer is anonymous, this array must not contain more than MAX_LENGTH_OF_UNIQUE_ID_IN_REQUEST items.
# Note that array is tail-optimized, i.e. it will not be prepended with length field.
#
uint8[<=16] unique_id

```

The following diagram may aid understanding of the allocatee side of the algorithm:  
下图可能有助于理解算法的被分配方（svg格式待修复）：

![Alt text](./picture/dynamic_node_id_allocatee_algorithm.svg)

#### Example （示例）
The following log provides a real-world example of a dynamic node ID allocation process:  
下面的日志提供了一个动态节点ID分配过程的真实例子：
```
Time   CAN ID     CAN data field
1.117  1EEE8100   01 44 C0 8B 63 5E 05 C0
1.117  1E000101   00 44 C0 8B 63 5E 05 C0
1.406  1EEBE500   00 F4 BC 10 96 DF 11 C1
1.406  1E000101   05 B0 00 44 C0 8B 63 81
1.406  1E000101   5E 05 F4 BC 10 96 DF 21
1.406  1E000101   11 41
1.485  1E41E100   00 A8 BA 54 47 C2
1.485  1E000101   29 BA FA 44 C0 8B 63 82
1.485  1E000101   5E 05 F4 BC 10 96 DF 22
1.485  1E000101   11 A8 BA 54 47 42
```

First, the allocatee waits for a random time interval in order to ensure that other allocations are not happening at the moment. After the delay, the allocatee announces its intention to get a node ID allocation by broadcasting the following anonymous message:  
首先，被分配器需要等待一个随机的时间间隔，以确保此时没有其他分配发生。延迟之后，被分配器将通过广播以下匿名消息来宣布其获得节点 ID 分配的意图:
```
1.117  1EEE8100   01 44 C0 8B 63 5E 05 C0
```

The allocator responds immediately with confirmation:  
分配器立即响应以确认：
```
1.117  1E000101   00 44 C0 8B 63 5E 05 C0
```

The allocatee waits for another random time interval in order to ensure that it will not conflict with other nodes that have unique node ID with the same first six bytes. After the delay, the allocatee sends the second-stage request:  
被分配器将再次等待一个随机时间间隔，以确保它不会与其他具有唯一节点 ID 的节点发生冲突，该节点的前 6 个字节相同。延迟后，被分配器发送第二阶段请求：
```
1.406  1EEBE500   00 F4 BC 10 96 DF 11 C1
```

The allocator responds immediately with confirmation. This time, the confirmation contains 12 bytes of unique ID, so it doesn’t fit one CAN frame, therefore the allocator resorts to a multi-frame transfer:  
分配器立即响应确认。这一次，确认包含 12 字节的唯一 ID，所以它无法一帧传完，因此分配器需要一个多帧传输：
```
1.406  1E000101   05 B0 00 44 C0 8B 63 81
1.406  1E000101   5E 05 F4 BC 10 96 DF 21
1.406  1E000101   11 41
```

The allocatee waits for another random time interval in order to ensure that it will not conflict with other nodes that have unique node ID with the same first twelve bytes. After the delay, the allocatee sends the third-stage request:  
被分配器再次等待一个随机时间间隔，以确保它不会与其他具有唯一节点 ID 的节点发生冲突，该节点的前 12 个字节相同。延迟后，被分配器发送第三阶段请求：
```
1.485  1E41E100   00 A8 BA 54 47 C2
```

At this moment the allocator has received full unique ID and the preferred node ID of the allocatee as well. The allocator can carry out the actual allocation and send a response:  
此时分配器已经收到完整的唯一 ID 并计算出了分配器首选节点 ID。分配器可以执行实际分配并发送响应：

```
1.485  1E000101   29 BA FA 44 C0 8B 63 82
1.485  1E000101   5E 05 F4 BC 10 96 DF 22
1.485  1E000101   11 A8 BA 54 47 42
```

This completes the process. Next time the allocatee sends an allocation request, it will be provided with the same node ID.  
这就完成了整个过程。下一次同一个被分配器发送分配请求时，它将被提供相同的节点ID。

The values used in the example above were the following:  
上述示例中使用的值如下：

| Name | Value |
| :--- | :--- |
|Unique ID | 44 C0 8B 63 5E 05 F4 BC 10 96 DF 11 A8 BA 54 47 (hex) |
|Preferred | node ID	0 (any) |
|Allocated | node ID	125 |

#### Non-redundant allocator (非冗余分配器)
UAVCAN does not impose specific requirements to the implementation of a non-redundant allocator, except its duties listed below.  
除了下面列出的功能外，UAVCAN 不会对非冗余分配器的实现做特定的要求。

#####Duties of the allocator （分配器的职责）
The allocator is tasked with monitoring the nodes present in the network. When a new node appears, the allocator must invoke uavcan.protocol.GetNodeInfo on it, and check the received unique ID against the allocation table. If a matching entry is not found in the table, the allocator will create one. If the node failed to respond to uavcan.protocol.GetNodeInfo after at least 3 attempts, the allocator will extend the allocation table with a mock entry, where the node ID is matching the real node ID of the non-responding node, and unique ID is set to zero. This ensures that dynamic nodes will not be granted a node ID value that is already taken by a static node. This requirement demonstrates why is it mandatory that dynamic nodes use the same unique ID both when responding to uavcan.protocol.GetNodeInfo and when publishing allocation requests.  
分配器的任务是监视网络中出现的节点。当出现新节点时，分配程序必须调用uavcan.protocol.GetNodeInfo，然后根据分配表检查接收到的唯一 ID。如果在表中没有找到匹配的条目，分配程序将创建一个新的。如果节点未能响应uavcan.protocol.GetNodeInfo，至少尝试3次之后，分配器将使用一个模拟条目扩展分配表，其中节点 ID 与没有响应的节点的实际节点 ID 匹配，唯一 ID 设置为 0 。

#### Redundant allocators （冗余分配器）
The algorithm used for replication of the allocation table across redundant allocators is a fairly direct implementation of the Raft consensus algorithm, as published in the paper “In Search of an Understandable Consensus Algorithm (Extended Version)” (Diego Ongaro and John Ousterhout). The following text assumes that the reader is familiar with the paper.  
用于在冗余分配器之间复制分配表的算法是由 Raft consensus 算法实现的，发表在论文“In Search of an Understandable Consensus Algorithm (Extended Version)” (Diego Ongaro and John Ousterhout)中。下面的文章假设读者已经读过这篇论文。

##### Raft log （筏项日志）
The Raft log contains entries of type uavcan.protocol.dynamic_node_id.server.Entry (defined below), where every entry contains Raft term number, unique ID, and the matching node ID value. Therefore, the raft log is the allocation table itself.  
筏日志包含 uavcan.protocol.dynamic_node_id.server.Entry 类型的条目（定义见下文），其中每个条目包含筏项编号、唯一 ID 和匹配的节点 ID 值。因此，筏项日志本身就是分配表。

Since the maximum number of entries in the allocation table is limited by the range of node ID, the log cannot contain more than 127 entries. Therefore, snapshot transfer and log compaction are not required, so they are not implemented in the algorithm.  
由于分配表中的最大条目数受到节点 ID 范围的限制，因此日志不能包含超过 127 个条目。由于快照传输和日志压缩不是必需的，因此它们在算法中没有实现。

When a server becomes the leader, it checks if the Raft log contains an entry for its own unique ID, and if it doesn’t, the leader adds its own allocation entry to the log. This feature guarantees that the raft log always contains at least one entry, therefore it is not necessary to support negative log indices, as proposed by the Raft paper.  
当一个服务器成为 leader 时，它会检查筏项日志中是否包含其自己的惟一 ID 条目，如果不包含，则 leader 将自己的分配条目添加到日志中。该特性保证了筏项日志始终包含至少一个条目，因此不需要像论文中建议的那样支持负的日志索引。

Since the log is write-only and limited in growth, all allocations are permanent. This restriction is acceptable, since UAVCAN is a vehicle bus, and configuration of vehicle’s components is not expected to change frequently. Old allocations can be removed in order to free node IDs for new allocations, by clearing the Raft log on all allocators.  
由于日志是只写的，并且增长有限，所以所有的分配都是永久性的。这个限制是可以接受的，因为 UAVCAN 是一种车辆总线，并且车辆组件的配置不希望频繁地改变。通过清除所有分配器上的筏项日志，可以删除旧的分配，以便为新的分配器释放节点 ID。

##### Cluster configuration （集群配置）
The allocators need to be aware of each other’s node ID in order to form a cluster. In order to learn each other’s node ID values, the allocators broadcast messages of type uavcan.protocol.dynamic_node_id.server.Discovery (defined below) until the cluster is fully discovered.  
为了形成集群，分配器之间需要知道彼此的节点 ID。为了获取彼此的节点 ID 值，分配器需要不断广播 uavcan.protocol.dynamic_node_id.server 类型的消息，直到集群被完整的发现。

This extension to the Raft algorithm makes the cluster almost configuration-free - the only parameter that must be configured on all servers of the cluster is the number of nodes in the cluster (everything else will be auto-detected).  
对 Raft算法的扩展使集群几乎不需要配置 —— 必须在集群的所有服务器上配置的惟一参数是集群中的节点数（其他所有参数都将自动检测）。

Runtime cluster membership changes are not supported, since they are not needed for a vehicle bus.  
不支持运行时集群成员关系更改，因为车辆总线不需要这么做。

#### Duties of the leader （领导者的职责）
The leader is tasked with monitoring the nodes present in the network. Please refer to the section dedicated to duties of a non-redundant allocator for details.  
领导者的任务是监控网络中的节点。有关详细信息，请参阅非冗余分配器职责一节。

Only the leader can process allocation requests and engage in communication with allocatees. An allocator is allowed to send allocation responses only if both conditions are met:  
只有领导者才能处理分配请求并与被分配者进行沟通。分配器只有在满足以下两个条件时才允许发送分配响应：

- The allocator is a leader.  
- 分配器是领导者。
- Its replica of the Raft log does not contain uncommitted entries (i.e. the last allocation request has been completed successfully).  
- 它的筏项日志副本不包含未提交的条目（即最后一个分配请求已成功完成）。

The second condition needs to be explained by an example.  
第二个条件需要用一个例子来解释。

Consider a case with two Raft nodes that are residing in different network partitions, unable to communicate with each other - A and B, both of them are leaders; A can commit to the log, and B is in a minor partition. Then there is an allocatee X that can exchange with both leaders, and an allocatee Y that can exchange only with A. Such a situation can occur as a result of a specific failure mode of redundant interfaces.  
考虑这样一种情况，两个筏项节点驻留在不同的网络分区中，无法相互通信 —— A 和 B 都是领导者；A 可以提交到日志，B 在一个小分区中。然后，有一个被分配器 X 可以与两个领导者同时进行交互，有一个被分配器 Y 只能与 A 交换。这种情况可能由冗余接口的特定故障模式造成。

Both allocatees X and Y initially send first-stage allocation requests; A responds to Y with a first-stage response, whereas B responds to X. Both X and Y will issue follow-up requests, which may cause A to mix allocation requests from different nodes, leading to reception of an invalid unique ID. When both leaders receive full unique ID values (A will receive an invalid one, and B will receive a valid unique ID of X), only A will be able to make a commit, because B is in a minor partition. Since both allocatees were unable to receive node ID values in this round, they will retry later.  
被分配者 X 和 Y 都会先发送第一阶段分配请求；A 响应 Y，而 B 响应 X。X 和 Y 都会发出后续的请求，这可能会导致 A 混合不同节点的分配请求，导致接收到一个无效的唯一ID。

Now, in order to prevent B from disrupting allocatee-allocator communication again, we introduce this second restriction: an allocator cannot exchange with allocatees as long as its log contains uncommitted entries.  
现在，为了防止 B 再次干扰分配器和被分配器之间的通信，我们引入第二个限制：只要分配器的日志包含未提交的条目，它就不能与被分配器进行交互。

Note that this restriction does not apply to allocation requests sent via CAN FD frames as these allow larger frames such that all necessary information to be exchanged in a single request and response. Only CAN FD can offer perfectly reliable allocation exchanges.  
注意，这个限制不适用于通过 CAN FD 帧发送的分配请求，因为它们允许在单个请求和响应中交互所有必要的信息。只有 CAN FD 才能提供完全可靠的分配交换。

__uavcan.protocol.dynamic_node_id.server.AppendEntries__
Default data type ID: 30
```
#
# THIS DEFINITION IS SUBJECT TO CHANGE.
#
# This type is a part of the Raft consensus algorithm.
# Please refer to the specification for details.
#

#
# Given min election timeout and cluster size, the maximum recommended request interval can be derived as follows:
#
#   max recommended request interval = (min election timeout) / 2 requests / (cluster size - 1)
#
# The equation assumes that the Leader requests one Follower at a time, so that there's at most one pending call
# at any moment. Such behavior is optimal as it creates uniform bus load, but it is actually implementation-specific.
# Obviously, request interval can be lower than that if needed, but higher values are not recommended as they may
# cause Followers to initiate premature elections in case of intensive frame losses or delays.
#
# Real timeout is randomized in the range (MIN, MAX], according to the Raft paper.
#
uint16 DEFAULT_MIN_ELECTION_TIMEOUT_MS = 2000
uint16 DEFAULT_MAX_ELECTION_TIMEOUT_MS = 4000

#
# Refer to the Raft paper for explanation.
#
uint32 term
uint32 prev_log_term
uint8 prev_log_index
uint8 leader_commit

#
# Worst-case replication time per Follower can be computed as:
#
#   worst replication time = (127 log entries) * (2 trips of next_index) * (request interval per Follower)
#
Entry[<=1] entries

---

#
# Refer to the Raft paper for explanation.
#
uint32 term
bool success

```

__uavcan.protocol.dynamic_node_id.server.RequestVote__
Default data type ID: 31
```
#
# THIS DEFINITION IS SUBJECT TO CHANGE.
#
# This type is a part of the Raft consensus algorithm.
# Please refer to the specification for details.
#

#
# Refer to the Raft paper for explanation.
#
uint32 term
uint32 last_log_term
uint8 last_log_index

---

#
# Refer to the Raft paper for explanation.
#
uint32 term
bool vote_granted

```

__uavcan.protocol.dynamic_node_id.server.Discovery__
Default data type ID: 390

```
#
# THIS DEFINITION IS SUBJECT TO CHANGE.
#
# This message is used by allocation servers to find each other's node ID.
# Please refer to the specification for details.
#
# A server should stop publishing this message as soon as it has discovered all other nodes in the cluster.
#
# An exception applies: when a server receives a Discovery message from another server where the list
# of known nodes is incomplete (i.e. len(known_nodes) < configured_cluster_size), the server must
# publish a discovery message once. This condition allows other servers to quickly re-discover the cluster
# after restart.
#

#
# This message should be broadcasted by the server at this interval until all other servers are discovered.
#
uint16 BROADCASTING_PERIOD_MS = 1000

#
# Number of servers in the cluster as configured on the sender.
#
uint8 configured_cluster_size

#
# Node ID of servers that are known to the publishing server, including the publishing server itself.
# Capacity of this array defines maximum size of the server cluster.
#
uint8[<=5] known_nodes
```

__uavcan.protocol.dynamic_node_id.server.Entry__
```
#
# THIS DEFINITION IS SUBJECT TO CHANGE.
#
# One dynamic node ID allocation entry.
# This type is a part of the Raft consensus algorithm.
# Please refer to the specification for details.
#

uint32 term             # Refer to the Raft paper for explanation.

uint8[16] unique_id     # Unique ID of this allocation.

void1
uint7 node_id           # Node ID of this allocation.

```

##### Example
The following log demonstrates relevant messages that were transferred over the CAN bus in the process of a dynamic node ID allocation for one allocatee, where the allocators were running a three-node Raft cluster. All node status messages were removed for clarity.  
下面的日志演示了在为一个被分配器分配动态节点 ID 的过程中，通过 CAN 总线传输的相关消息，其中分配器运行一个三节点筏项集群。为了清晰起见，删除了所有节点状态消息。

The configuration was as follows:
配置如下：

| Name | Value |
| :--- | :--- |
| Number of allocators	| 3 |
| Allocators’ node ID | 1, 2, 3 |
| Leader’s node ID | 1 |
| Allocatee’s unique ID | 44 C0 8B 63 5E 05 F4 BC 83 3B 3A 88 1C 43 60 50 (hex) |
| Preferred node ID |	0 (any) |
| Allocated node ID |	125 |
| Discovery broadcasting interval |	1 second |
| AppendEntries interval | 1 second per follower |

```
Time   CAN ID     CAN data field
0.000  1E018601   03 01 C0
0.512  1E018602   03 02 01 C0
0.905  1E018603   03 03 01 02 C0
1.000  1E018601   03 01 02 03 C1
1.512  1E018602   03 02 01 03 C1
<cluster maintenance traffic omitted for clarity>
2.569  1EEE8100   01 44 C0 8B 63 5E 05 C0
2.569  1E000101   00 44 C0 8B 63 5E 05 C0
2.684  1E238D00   00 F4 BC 83 3B 3A 88 C1
2.684  1E000101   5C EF 00 44 C0 8B 63 81
2.684  1E000101   5E 05 F4 BC 83 3B 3A 21
2.684  1E000101   88 41
2.756  1E1E8381   5F CF 2E 00 00 00 04 85
2.756  1E1E8381   00 00 00 05 05 65
2.756  1E1E0183   2E 00 00 00 80 C5
2.871  1E63ED00   00 1C 43 60 50 C2
3.256  1E1E8281   9C 38 2E 00 00 00 04 87
3.256  1E1E8281   00 00 00 05 05 2E 00 27
3.256  1E1E8281   00 00 44 C0 8B 63 5E 07
3.256  1E1E8281   05 F4 BC 83 3B 3A 88 27
3.256  1E1E8281   1C 43 60 50 7D 47
3.258  1E1E0182   2E 00 00 00 80 C7
3.563  1E2F0D00   01 44 C0 8B 63 5E 05 C3
3.756  1E1E8381   9C 38 2E 00 00 00 04 86
3.756  1E1E8381   00 00 00 05 05 2E 00 26
3.756  1E1E8381   00 00 44 C0 8B 63 5E 06
3.756  1E1E8381   05 F4 BC 83 3B 3A 88 26
3.756  1E1E8381   1C 43 60 50 7D 46
3.756  1E000101   C7 36 FA 44 C0 8B 63 82
3.756  1E000101   5E 05 F4 BC 83 3B 3A 22
3.756  1E000101   88 1C 43 60 50 42
3.758  1E1E0183   2E 00 00 00 80 C6
4.256  1E1E8281   65 19 2E 00 00 00 2E 88
4.256  1E1E8281   00 00 00 06 06 68
4.256  1E1E0182   2E 00 00 00 80 C8
4.756  1E1E8381   65 19 2E 00 00 00 2E 87
4.756  1E1E8381   00 00 00 06 06 67
4.756  1E1E0183   2E 00 00 00 80 C7
```

Once the first node of the cluster has been started, it has published a cluster discovery message so it could become aware of its siblings, and other two nodes that were started a fraction of a second later did the same:  
一旦启动了集群的第一个节点，它就会发布一个集群发现消息，这样它就可以知道它的兄弟节点，而其他两个稍后启动的节点也会这样做：
```
0.000  1E018601   03 01 C0

0.512  1E018602   03 02 01 C0

0.905  1E018603   03 03 01 02 C0

1.000  1E018601   03 01 02 03 C1

1.512  1E018602   03 02 01 03 C1
```

It can be seen that the last two discovery messages contain complete list of all nodes in the cluster. The allocators have detected the fact that all nodes in the cluster were now aware of each other, and ceased to broadcast discovery messages in order to not pollute the bus with redundant traffic.  
可以看到，最后两条发现消息包含集群中所有节点的完整列表。分配器已经检测到集群中的所有节点现在都知道彼此的存在，并且停止广播发现消息，以避免用冗余的流量污染总线。

Afterwards, the allocators ran elections and have elected the node 1 as their leader. The leader then performed a few AppendEntries calls in order to synchronize the replicated log. These exchanges are not shown for the sake of clarity.  
随后，分配器进行了选举，选出节点 1 为他们的领导者。然后，领导者执行几个附录条目调用，以便同步复制的日志。这些交互并没有被清晰呈现在上文中。

The allocatee has appeared on the bus and has published first-stage and second-stage allocation requests. The current leader was in charge with communicating with allocatee, other two allocators were silent:  
被分配器已经出现在总线上，并发布了第一阶段和第二阶段的分配请求。现任领导者负责与被分配者沟通，其他两名分配者沉默：
```
2.569  1EEE8100   01 44 C0 8B 63 5E 05 C0   <-- First stage request

2.569  1E000101   00 44 C0 8B 63 5E 05 C0   <-- First stage response

2.684  1E238D00   00 F4 BC 83 3B 3A 88 C1   <-- Second stage request

2.684  1E000101   5C EF 00 44 C0 8B 63 81   <-- Second stage response
2.684  1E000101   5E 05 F4 BC 83 3B 3A 21
2.684  1E000101   88 41
```

While the allocatee was waiting for expiration of the random timeout, the leader has performed a keep-alive AppendEntries call to the allocator 3:  
当被分配器在等待中超时了，领导者会对分配器 3 执行一个 keep-alive 的专项调用：
```
2.756  1E1E8381   5F CF 2E 00 00 00 04 85   <-- Empty AppendEntries request
2.756  1E1E8381   00 00 00 05 05 65

2.756  1E1E0183   2E 00 00 00 80 C5         <-- AppendEntries response
```

Then the allocatee has broadcasted the third-stage allocation request:  
然后被分配器广播第三阶段分配请求：
```
2.871  1E63ED00   00 1C 43 60 50 C2
```

At this point the leader had the full unique ID of the allocatee, so it has started the process of allocation and log replication. In order to complete the allocation, the leader had to replicate the new entry of the Raft log to a majority of allocators (see the Raft paper for details):  
此时，领导者拥有被分配器完整的唯一 ID，因此它开始了分配和日志复制的过程。为了完成分配，领导者必须将筏项日志的新条目复制给其他分配器（详见 筏项算法的论文）:

```
3.256  1E1E8281   9C 38 2E 00 00 00 04 87   <-- AppendEntries request with new allocation
3.256  1E1E8281   00 00 00 05 05 2E 00 27
3.256  1E1E8281   00 00 44 C0 8B 63 5E 07
3.256  1E1E8281   05 F4 BC 83 3B 3A 88 27
3.256  1E1E8281   1C 43 60 50 7D 47

3.258  1E1E0182   2E 00 00 00 80 C7         <-- AppendEntries response with confirmation
```

It can be seen that the follower took 2 milliseconds to update its persistent storage.  
可以看出其他的分配器花费了 2 毫秒来更新它的存储。

While the leader was busy replicating the allocation table (it could not complete the allocation until the new log entry was committed), the allocatee has given up waiting for a response and decided to restart the process. This is not an error condition, but a normal behavior. This time the leader did not engage in communication with the allocatee, because the Raft log contained uncommitted entries.  
当领导者忙于复制分配表时（在提交新的日志条目之前它无法完成分配），另一个被分配器已经放弃等待响应并决定重新启动进程。这不是一个错误状态，而是一个正常的行为。这时领导者没有与被分配器进行通信，因为木筏日志包含未提交的条目。
```
3.563  1E2F0D00   01 44 C0 8B 63 5E 05 C3   <-- First stage request, no response from the leader
```

Some time later the leader decided to replicate the new log entry to the other follower:  
一段时间后，领导者决定复制新的日志条目给另一个分配器：
```
3.756  1E1E8381   9C 38 2E 00 00 00 04 86   <-- AppendEntries request with new allocation
3.756  1E1E8381   00 00 00 05 05 2E 00 26
3.756  1E1E8381   00 00 44 C0 8B 63 5E 06
3.756  1E1E8381   05 F4 BC 83 3B 3A 88 26
3.756  1E1E8381   1C 43 60 50 7D 46
```

Immediately afterwards, the leader has noticed that the new entry has already been replicated to a majority of allocators, therefore (see the Raft paper) the commit index could be incremented, which completed the allocation. Having detected that, the leader has published the allocation response:  
紧接着，领导者注意到新的条目已经被复制到大多数分配器中，因此提交增加索引，从而完成分配。检测以上动作后，领导者发布了分配响应：
```
3.756  1E000101   C7 36 FA 44 C0 8B 63 82
3.756  1E000101   5E 05 F4 BC 83 3B 3A 22
3.756  1E000101   88 1C 43 60 50 42
```
While the leader was engaged in communications with the allocatee, the follower 3 has finished updating its persistent storage and responded with confirmation:  
当领导者与被分配器 3 进行交流时，分配器 3 已经完成了对其持久存储的更新，并回复了确认：
```
3.758  1E1E0183   2E 00 00 00 80 C6
```
At this moment the process was finished. The leader then continued to invoke keep-alive AppendEntries calls to the followers:  
这时，整个过程结束了。然后，领导者继续向其他的分配器调用 keep-alive 附录条目:
```
4.256  1E1E8281   65 19 2E 00 00 00 2E 88   <-- Empty AppendEntries request
4.256  1E1E8281   00 00 00 06 06 68

4.256  1E1E0182   2E 00 00 00 80 C8         <-- AppendEntries response

4.756  1E1E8381   65 19 2E 00 00 00 2E 87   <-- Empty AppendEntries request
4.756  1E1E8381   00 00 00 06 06 67

4.756  1E1E0183   2E 00 00 00 80 C7         <-- AppendEntries response
```


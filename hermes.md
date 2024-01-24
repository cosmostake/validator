[[chains]]

id = 'umee-1'
ccv_consumer_chain = false
rpc_addr = 'http://127.0.0.1:21657'
grpc_addr = 'http://127.0.0.1:9094'
event_source = { mode = 'push', url = 'ws://127.0.0.1:21657/websocket', batch_delay = '500ms' }
rpc_timeout = '20s'
trusted_node = false
account_prefix = 'umee'
key_name = 'cosmostake2'
address_type = { derivation = 'cosmos' }
store_prefix = 'ibc'
default_gas = 400000
max_gas = 10000000
gas_price = { price = 0.1, denom = 'uumee' }
gas_multiplier = 1.2
max_msg_num = 30
max_tx_size = 2097152
clock_drift = '15s'
max_block_time = '30s'
trusting_period = '7days'
trust_threshold = { numerator = '1', denominator = '3' }
memo_prefix = 'You have been relayed by COSMØSTAKE'
[chains.packet_filter]
policy = 'allow'
list = [
#   ['transfer', 'channel-43'],
# osmosis
   ['transfer', 'channel-0'],
#   ['transfer', 'channel-34'],
# Persistence
   ['transfer', 'channel-51'],
#   ['transfer', 'channel-53'],
# Juno
   ['transfer', 'channel-2'],
]

[[chains]]
id = 'osmosis-1'
rpc_addr = 'http://127.0.0.1:26657'
grpc_addr = 'http://127.0.0.1:9090'
event_source = { mode = 'push', url = 'ws://127.0.0.1:26657/websocket', batch_delay = '500ms' }
rpc_timeout = '10s'
trusted_node = false
account_prefix = 'osmo'
key_name = 'cosmostake5'
address_type = { derivation = 'cosmos' }
store_prefix = 'ibc'
gas_price = { price = 0.0026, denom = 'uosmo' }
default_gas = 500000
max_gas = 120000000
gas_multiplier = 1.4
max_msg_num = 30
max_tx_size = 1800000
clock_drift = '15s'
max_block_time = '30s'
trusting_period = '7days'
trust_threshold = { numerator = '1', denominator = '3' }
memo_prefix = 'You have been relayed by COSMØSTAKE'
[chains.packet_filter]
policy = 'allow'
list = [
#   ['ica*', '*'],
# umee
   ['transfer', 'channel-184'],
#   ['transfer', 'channel-42'],
#   ['transfer', 'channel-229'],
# Persistence
   ['transfer', 'channel-4'],
# Kichain
   ['transfer', 'channel-77'],
# Juno
   ['transfer', 'channel-42'],
]

[[chains]]
id = 'core-1'
rpc_addr = 'http://127.0.0.1:23657'
grpc_addr = 'http://127.0.0.1:9098'
event_source = { mode = 'push', url = 'ws://127.0.0.1:23657/websocket', batch_delay = '500ms' }
rpc_timeout = '10s'
trusted_node = false
account_prefix = 'persistence'
key_name = 'cosmostake7'
address_type = { derivation = 'cosmos' }
store_prefix = 'ibc'
gas_price = { price = 0.01, denom = 'uxprt' }
default_gas = 600000
max_gas = 15000000
gas_multiplier = 1.2
max_msg_num = 30
max_tx_size = 1800000
clock_drift = '15s'
max_block_time = '30s'
trusting_period = '14days'
trust_threshold = { numerator = '1', denominator = '3' }
memo_prefix = 'You have been relayed by COSMØSTAKE'
[chains.packet_filter]
policy = 'allow'
list = [
#   ['ica*', '*'],
# Osmosis
['transfer', 'channel-6'],
# Umee
['transfer', 'channel-81'],
# Juno
['transfer', 'channel-37'],
]

[[chains]]
id = 'kichain-2'
rpc_addr = 'http://127.0.0.1:24657'
grpc_addr = 'http://127.0.0.1:9088'
event_source = { mode = 'push', url = 'ws://127.0.0.1:24657/websocket', batch_delay = '500ms' }
rpc_timeout = '20s'
account_prefix = 'ki'
key_name = 'cosmostake9'
store_prefix = 'ibc'
address_type = { derivation = 'cosmos' }
default_gas = 300000
max_gas = 3500000
gas_price = { price = 0.025, denom = 'uxki' }
gas_multiplier = 1.4
max_msg_num = 30
max_tx_size = 1800000
clock_drift = '15s'
max_block_time = '10s'
trusting_period = '7days'
memo_prefix = 'You have been relayed by COSMOSTAKE'
trust_threshold = { numerator = '1', denominator = '3' }
[chains.packet_filter]
policy = 'allow'
list = [
# Osmosis
  ['transfer', 'channel-0'],
# Juno
  ['transfer', 'channel-8'],
]

[[chains]]
id = 'juno-1'
rpc_addr = 'http://127.0.0.1:20657'
grpc_addr = 'http://127.0.0.1:9092'
event_source = { mode = 'push', url = 'ws://127.0.0.1:20657/websocket', batch_delay = '500ms' }
rpc_timeout = '10s'
account_prefix = 'juno'
key_name = 'cosmostake'
store_prefix = 'ibc'
address_type = { derivation = 'cosmos' }
default_gas = 300000
max_gas = 1000000
gas_price = { price = 0.0025, denom = 'ujuno' }
gas_multiplier = 1.4
max_msg_num = 30
max_tx_size = 1800000
clock_drift = '15s'
max_block_time = '10s'
trusting_period = '14days'
memo_prefix = 'You have been relayed by COSMOSTAKE'
trust_threshold = { numerator = '1', denominator = '3' }
[chains.packet_filter]
policy = 'allow'
list = [
# Osmosis
  ['transfer', 'channel-0'],
# Persistence
  ['transfer', 'channel-33'],
# Umee
  ['transfer', 'channel-62'],
# Kichain
  ['transfer', 'channel-58'],
]


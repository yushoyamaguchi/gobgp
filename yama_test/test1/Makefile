sudo-in-advance:
	sudo pwd

setup: sudo-in-advance
	-$(MAKE) set-netns || true
	-$(MAKE) build-gobgp || true
	-$(MAKE) run-gobgp || true
	-$(MAKE) gobgp-static-route || true

cleanup: sudo-in-advance
	-$(MAKE) kill-gobgp || true
	-$(MAKE) del-netns || true

set-netns:
	sudo ip netns add host1
	sudo ip netns exec host1 ip link set lo up
	sudo ip link add name veth1 type veth peer name host1-veth1
	sudo ip link set host1-veth1 netns host1
	sudo ip addr add 10.0.0.1/24 dev veth1
	sudo ip link set veth1 up
	sudo ip netns exec host1 ip addr add 10.0.0.2/24 dev host1-veth1
	sudo ip netns exec host1 ip link set host1-veth1 up

del-netns:
	sudo ip link del veth1 || true
	sudo ip netns del host1 || true

build-gobgp:
	go mod download
	go build ../../cmd/gobgp
	go build ../../cmd/gobgpd

run-gobgp:
	sudo ./gobgpd -f ./host.conf > ./host.log 2>&1 &
	sudo ip netns exec host1 ./gobgpd -f ./netns.conf > ./netns.log 2>&1 &

gobgp-static-route:
	sleep 2
	./gobgp global rib add 10.0.0.0/24  nexthop 0.0.0.0 || true
	./gobgp global rib add 192.168.1.0/24 nexthop 0.0.0.0 || true
	sudo ip netns exec host1 ./gobgp global rib add 10.0.0.0/24  nexthop 0.0.0.0 || true
	sudo ip netns exec host1 ./gobgp global rib add 192.168.2.0/24 nexthop 0.0.0.0 || true

kill-gobgp:
	sudo pkill gobgpd

verify-routes:
	@echo "=== Host BGP routes ==="
	./gobgp global rib
	@echo "=== Netns BGP routes ==="
	sudo ip netns exec host1 ./gobgp global rib

show-neighbors:
	@echo "=== Host BGP neighbors ==="
	./gobgp neighbor
	@echo "=== Netns BGP neighbors ==="
	sudo ip netns exec host1 ./gobgp neighbor
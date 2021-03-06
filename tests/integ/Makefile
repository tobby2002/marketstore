################################################################################
# Commands to automate integration tests
################################################################################

LATEST_TAG ?= $(shell bin/dockertags alpacamarkets/marketstore | tail -1)
IMAGE_NAME ?= alpacamarkets/marketstore:${LATEST_TAG}
CONTAINER_NAME = integration_tests_mstore

# User targets
################################################################################
connect: run
	@curl -s --data-binary '{"jsonrpc":"2.0", "method":"DataService.ListSymbols", "id":1, "params": {"parameters": {}}}' -H 'Content-Type: application/json' http://localhost:5993/rpc ; if [ $$? -ne 0 ]; then echo "Failed"; else echo "Passed"; fi


run: _init
	@if [ `bin/check_running ${CONTAINER_NAME}` -eq 0 ]; then $(MAKE) _startup; fi

stop:
	-if [ `bin/check_running ${CONTAINER_NAME}` -eq 1 ]; then \
		docker stop ${CONTAINER_NAME}; fi
	-docker rm -f ${CONTAINER_NAME}

test:
	@bin/runtests.sh

clean: stop
	rm -rf data tests/__pycache__

# Utils
################################################################################
_init:
	@if [ ! -d data/mktsdb ]; then rm -rf data; $(MAKE) _get_data; fi

_get_data:
	@rm -rf data && mkdir data
	@if [ ! -f /tmp/mktsdb.tgz ]; then wget https://s3.amazonaws.com/dev.alpaca.markets/gobroker/mktsdb.tar.gz -O /tmp/mktsdb.tgz; fi
	@tar -C data -xzf /tmp/mktsdb.tgz


_startup: stop
	docker run --name ${CONTAINER_NAME} -d -p 5993:5993 -v $(CURDIR):/project -w /project $(IMAGE_NAME) \
		start --config /project/bin/mkts.yml
	@sleep 2
	@if [ `bin/check_running ${CONTAINER_NAME}` -eq 0 ]; then \
		echo "Failed to start a marketstore instance"; false; fi



# Tests
################################################################################
_start_pyclient_container:
	make -C dockerfiles/pyclient rm build run

test_driver: clean run _start_pyclient_container
	TEST_FILENAME='/project/tests/$@.py' make -C dockerfiles/pyclient test

test_not_empty: clean run _start_pyclient_container
	TEST_FILENAME='/project/tests/$@.py' make -C dockerfiles/pyclient test

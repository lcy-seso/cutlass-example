.PHONY: all clean run

all:
	mkdir -p build
	cd build && cmake .. && make

run: all
	cd build && ./hopper_grouped_gemm

clean:
	rm -rf build 

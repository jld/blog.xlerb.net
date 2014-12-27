SRCS=$(wildcard .frogrc _src/*.html _src/*.md _src/posts/*.md)
targets=westworld.xlerb.net:/home/www/blog undertow.xlerb.net:/home/www/blog

all: .build_stamp
.build_stamp: $(SRCS) out
	raco frog -b
	touch .build_stamp

out:
	mkdir out
	for n in css fonts img js; do \
	  [ -e "out/$n" ] && continue; \
	  ln -s "../$n" "out/$n" || break; \
	done

deploy: all
	for target in $(targets); do \
	  rsync -HPav --copy-unsafe-links --exclude '*~' out/ "$${target}/"; \
	done

clean:
	raco frog -c

cleaner:
	rm -r out .frog

.PHONY: all clean cleaner

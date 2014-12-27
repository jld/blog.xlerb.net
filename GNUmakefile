SRCS=$(wildcard .frogrc _src/*.html _src/*.md _src/posts/*.md)

deployments=westworld undertow
remote_westworld=westworld.xlerb.net:/home/www/blog
remote_undertow=undertow.xlerb.net:/home/www/blog

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

deploy:
	$(MAKE) -j$(words $(deployments)) parallel_deploy

parallel_deploy: $(foreach name,$(deployments),deploy_$(name))

define deploy__template =
deploy_$(1): .build_stamp
	rsync -HPav --copy-unsafe-links --exclude '*~' out/ $$(remote_$(1))
endef

$(foreach name,$(deployments),$(eval $(call deploy__template,$(name))))

clean:
	raco frog -c

cleaner:
	rm -r out .frog

.PHONY: all clean cleaner $(foreach name,$(deployments),deploy_$(name))

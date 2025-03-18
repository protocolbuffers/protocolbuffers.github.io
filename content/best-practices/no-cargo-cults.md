+++
title = "Avoid Cargo Culting"
weight = 90
description = "Avoid using features where they are not needed."
type = "docs"
+++

Do not
[cargo cult](https://en.wikipedia.org/wiki/Cargo_cult_programming)
settings in proto files. If \
you are creating a new proto file based on existing schema definitions, don't
apply option settings except for those that you understand the need for.

## Best Practices Specific to Editions {#editions}

Avoid applying [editions features](/editions/features)
except when they're actually necessary. Features in `.proto` files signal the
use of either experimental future behaviors or deprecated past behaviors. Best
practices for the latest edition will always be the default. New proto schema
definition content should remain feature-free, except if you want to early-adopt
a feature for future behavior that's being rolled out.

Copying-forward feature settings without understanding why they are set can lead
to unexpected behaviors in your code.

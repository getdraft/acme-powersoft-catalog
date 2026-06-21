# Onboarding: Infra & DevOps Team

Sofia -- two things to catalog from `acme-powersoft-infra-devops-product`:
the reusable Terraform modules (these aren't deployable objects at all --
note that distinction explicitly) and the admin console itself, which IS
a real RuntimeService like everyone else's product.

## Admin follow-up

Confirmed, Sofia -- "answer honestly with a lower bar" is the intended
pattern, not a missing activation tier. A RequirementGroup with
`requirementMode: mandatory` means the question must be answered, not that
the answer must hit a fixed bar. 99% with fixed replicas is a fully valid
answer for an internal tool. That said: your instinct that this *feels*
like it should be a different activation tier is worth recording, since
"mandatory-but-the-bar-can-vary" and "lower-tier-with-a-different-bar"
produce the same catalog content today but very different audit/reporting
semantics later. Going in the EXERCISE_REPORT as a v1.1 consideration, not
a v1.0 blocker.

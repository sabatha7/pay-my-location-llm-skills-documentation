{
  "version": "1.0",

  "actions": {
    "create_transfer": {
      "route": "/transfer/create",
      "method": "POST",
      "required_args": ["from", "to", "amount"],
      "produces": ["transferId"],
      "description": "Create a transfer that requires approval"
    },

    "approve_transfer": {
      "route": "/transfer/approve",
      "method": "POST",
      "required_args": ["transferId"],
      "description": "Approve a pending transfer",
      "requires": [
        {
          "resource": "2fa",
          "mode": "blocking",
          "execution": "precondition",
          "bind_to": "approve_transfer"
        }
      ],
      "constraints": [
        "must_be_owner_or_admin",
        "must_be_pending"
      ]
    },

    "dispute_transfer": {
      "route": "/transfer/dispute",
      "method": "POST",
      "required_args": ["transferId", "reason"],
      "constraints": [
        "must_be_participant_or_admin"
      ]
    },

    "file_for_an_approve_proceeding": {
      "route": "/transfer/subscribe",
      "method": "POST",
      "required_args": ["transferId", "note"],
      "description": "Submit proof of conveyance",
      "constraints": [
        "must_be_buyer"
      ]
    }
  },

  "resources": {
    "2fa": {
      "type": "verification",
      "mode": "challenge_response",
      "blocking": true,

      "actions": {
        "send": {
          "route": "/2fa/send",
          "method": "POST",
          "required_args": ["userId", "context"],
          "produces": ["twoFactorId"],
          "callable_by": "system_only"
        },

        "verify": {
          "route": "/2fa/verify",
          "method": "POST",
          "required_args": ["userId", "twoFactorId", "code"],
          "constraints": [
            "must_match_context",
            "must_not_be_expired",
            "single_use_only"
          ],
          "callable_by": "system_after_user_input"
        }
      },

      "binding_rules": {
        "type": "strict",
        "rules": [
          "2fa_token_bound_to_context",
          "cannot_be_reused_across_intents",
          "expires_after_short_duration",
          "invalidates_on_success"
        ]
      }
    }
  },

  "execution_model": {
    "state": {
      "intent": "string",
      "args": {},
      "status": "pending | awaiting_2fa | ready | executing | completed | failed",

      "resources": {
        "2fa": {
          "required": "boolean",
          "status": "not_started | code_sent | awaiting_input | verified | failed",
          "twoFactorId": "string | null",
          "attempts": "number",
          "max_attempts": 3
        }
      }
    }
  },

  "orchestrator_rules": [
    {
      "condition": "intent.requires.resource == 2fa AND state.resources.2fa.status == not_started",
      "action": "call 2fa.send",
      "next_state": "awaiting_2fa"
    },
    {
      "condition": "state.resources.2fa.status == code_sent",
      "action": "prompt_user_for_code"
    },
    {
      "condition": "user_provides_code",
      "action": "call 2fa.verify"
    },
    {
      "condition": "2fa.verify.success == true",
      "action": "mark_2fa_verified",
      "next_state": "ready"
    },
    {
      "condition": "state.status == ready",
      "action": "execute_intent_action"
    },
    {
      "condition": "2fa.verify.failure AND attempts < max_attempts",
      "action": "retry_prompt"
    },
    {
      "condition": "2fa.verify.failure AND attempts >= max_attempts",
      "action": "fail_execution"
    }
  ],

  "llm_contract": {
    "allowed_actions": [
      "intent_detection",
      "argument_extraction",
      "user_interaction"
    ],
    "forbidden_actions": [
      "direct_2fa_send_call",
      "direct_2fa_verify_call",
      "bypassing_required_resources"
    ],
    "responsibilities": [
      "detect user intent",
      "collect required arguments",
      "prompt user for missing data",
      "handle 2fa code input interaction"
    ]
  }
}

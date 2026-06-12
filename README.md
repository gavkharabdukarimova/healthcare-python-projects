import streamlit as st

st.set_page_config(
    page_title="Healthcare Pre-Pricing Estimator",
    page_icon="🏥",
    layout="centered"
)

st.title("Healthcare Pre-Pricing Estimator")
st.write("Estimate your possible out-of-pocket cost before visiting a healthcare provider.")

st.warning(
    "This tool is for educational and estimation purposes only. "
    "It does not replace official insurance verification, provider billing, or medical advice."
)

st.header("Patient Insurance Information")

insurance_plan = st.text_input("Insurance Plan Name", placeholder="Example: Blue Cross PPO")
member_id = st.text_input("Member ID", placeholder="Optional for demo")

deductible = st.number_input(
    "Annual Deductible Amount ($)",
    min_value=0.0,
    value=1500.0,
    step=50.0
)

deductible_met = st.number_input(
    "Deductible Already Met ($)",
    min_value=0.0,
    value=300.0,
    step=50.0
)

copay = st.number_input(
    "Copay for Visit ($)",
    min_value=0.0,
    value=40.0,
    step=5.0
)

coinsurance = st.slider(
    "Coinsurance After Deductible (%)",
    min_value=0,
    max_value=100,
    value=20
)

out_of_pocket_max = st.number_input(
    "Out-of-Pocket Maximum ($)",
    min_value=0.0,
    value=6000.0,
    step=100.0
)

oop_met = st.number_input(
    "Out-of-Pocket Already Met ($)",
    min_value=0.0,
    value=500.0,
    step=50.0
)

st.header("Visit Information")

visit_type = st.selectbox(
    "Select Visit Type",
    [
        "Primary Care Visit",
        "Specialist Visit",
        "Urgent Care Visit",
        "Emergency Room Visit",
        "Lab Test",
        "Imaging / X-Ray",
        "MRI / CT Scan"
    ]
)

estimated_provider_price = st.number_input(
    "Estimated Provider Price Before Insurance ($)",
    min_value=0.0,
    value=250.0,
    step=25.0
)

network_status = st.selectbox(
    "Provider Network Status",
    ["In-Network", "Out-of-Network"]
)

def calculate_patient_cost(
    provider_price,
    deductible,
    deductible_met,
    copay,
    coinsurance,
    oop_max,
    oop_met,
    network_status
):
    remaining_deductible = max(deductible - deductible_met, 0)
    remaining_oop = max(oop_max - oop_met, 0)

    adjusted_price = provider_price

    if network_status == "Out-of-Network":
        adjusted_price = provider_price * 1.5

    deductible_payment = min(adjusted_price, remaining_deductible)
    remaining_after_deductible = max(adjusted_price - deductible_payment, 0)

    coinsurance_payment = remaining_after_deductible * (coinsurance / 100)

    estimated_patient_cost = copay + deductible_payment + coinsurance_payment

    final_patient_cost = min(estimated_patient_cost, remaining_oop)

    insurance_estimated_payment = max(adjusted_price - final_patient_cost, 0)

    return {
        "adjusted_price": adjusted_price,
        "remaining_deductible": remaining_deductible,
        "deductible_payment": deductible_payment,
        "coinsurance_payment": coinsurance_payment,
        "copay": copay,
        "estimated_patient_cost": final_patient_cost,
        "insurance_estimated_payment": insurance_estimated_payment,
        "remaining_oop": remaining_oop
    }

if st.button("Calculate Estimated Cost"):
    result = calculate_patient_cost(
        estimated_provider_price,
        deductible,
        deductible_met,
        copay,
        coinsurance,
        out_of_pocket_max,
        oop_met,
        network_status
    )

    st.subheader("Estimated Cost Summary")

    st.metric("Estimated Patient Cost", f"${result['estimated_patient_cost']:.2f}")
    st.metric("Estimated Insurance Payment", f"${result['insurance_estimated_payment']:.2f}")
    st.metric("Adjusted Provider Price", f"${result['adjusted_price']:.2f}")

    st.write("Cost Breakdown")
    st.write(f"Copay: ${result['copay']:.2f}")
    st.write(f"Remaining Deductible: ${result['remaining_deductible']:.2f}")
    st.write(f"Applied to Deductible: ${result['deductible_payment']:.2f}")
    st.write(f"Coinsurance Payment: ${result['coinsurance_payment']:.2f}")
    st.write(f"Remaining Out-of-Pocket Limit: ${result['remaining_oop']:.2f}")

    st.info(
        "Final medical bills may be different because actual prices depend on CPT codes, "
        "provider contracts, insurance eligibility, prior authorization, diagnosis codes, "
        "medical necessity, and claim processing rules."
    )

st.header("Future Version Features")
st.write(
    "Future versions may allow users to upload an insurance card, extract payer information, "
    "connect CPT/HCPCS codes, verify eligibility, and generate a more accurate pre-visit estimate."
)

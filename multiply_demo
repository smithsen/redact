use ff::PrimeField;
use bellman::{Circuit, ConstraintSystem, SynthesisError};
use rand::rngs::OsRng;
use ff::Field;
use bls12_381::{Bls12, Scalar};
use bellman::groth16::{
    create_random_proof, generate_random_parameters, prepare_verifying_key, verify_proof,
    Proof,
};
use std::time::{Duration,Instant};
// for LEN = 100; params: 1.9sec, proof: 1.6 sec, for verification: 0.7sec
// for LEN = 500; params: 8.6sec, proof: 4.85 sec, for verification: 3.3 sec
// for LEN = 1000; params: 16sec, proof: 8.3sec, for verification:  6.6sec
// for LEN = 5000; params: 89.5sec, proof: 38.3sec, for verification: 33.16sec 
const LEN: usize = 1000;
struct MultiplyDemo<S: PrimeField>{
	a: Option<[S;LEN]>,
	b: Option<[S;LEN]>,
	w: Option<[S;LEN]>,
}

fn main(){
	println!("Parameter Generation...");
	let mut time_param = Duration::new(0,0);
	let mut time_prove = Duration::new(0,0);
	let mut time_verify = Duration::new(0,0);
	let start = Instant::now();	
	let params = {
			let c = MultiplyDemo{
				a: None,
				b: None,
				w: None,						
				};
		     generate_random_parameters::<Bls12,_,_>(c,&mut OsRng).unwrap()
		 };
	time_param += start.elapsed();
	let time_param_f =
        	 time_param.subsec_nanos() as f64 / 1_000_000_000f64 + (time_param.as_secs() as f64);

	println!("Parameter generation time: {:?}",time_param_f);
	let pvk = prepare_verifying_key(&params.vk);
	//println!("{:?}",params);
	//println!("{:?}",pvk);
	let a_prior: u8 = 10;
	let b_prior: u8 = 20;
	//let c_prior: u8 = 200;
	let a: [Scalar;LEN] = [convert_u8(a_prior);LEN];
	//println!("private input: {:?}",a);
	let b: [Scalar;LEN] = [convert_u8(b_prior);LEN];
	//println!("private input: {:?}",b);
	let w_prior = a_prior*b_prior;
	let w: [Scalar;LEN] = [convert_u8(w_prior);LEN];
	let value: [Scalar;LEN] = [convert_u8(w_prior);LEN];
	//println!("public input: {:?}",value);
	//let d = w;
	//let c = pairing::Engine::Fr::c_prior;
	let c = MultiplyDemo{
		a: Some(a),
		b: Some(b),
		w: Some(w),
	};
	//println!("{:?}",c);
	println!("Creating Proof Elements..");
	let start_proof = Instant::now();
    	let proof = create_random_proof(c, &params, &mut OsRng).unwrap();
	time_prove += start_proof.elapsed();
	let time_prove_f =
        	 time_prove.subsec_nanos() as f64 / 1_000_000_000f64 + (time_prove.as_secs() as f64);

	println!("Time after proof generation: {:?}",time_prove_f);	

	println!("Verifying...");
	let start_verify = Instant::now();
    	assert!(verify_proof(&pvk,&proof,&value).is_ok());
	time_verify += start_verify.elapsed();
	let time_verify_f =
        	 time_verify.subsec_nanos() as f64 / 1_000_000_000f64 + (time_verify.as_secs() as f64);

	println!("Time after verification: {:?}",time_verify_f);	

}



fn convert_u8<S: ff::PrimeField>(x: u8) -> S {
    S::from(u64::from(x))
}

//implementation of the demo circuit 
impl <S: PrimeField> Circuit<S>  for MultiplyDemo<S>{
  fn synthesize<CS: ConstraintSystem<S>>(self, cs: &mut CS) -> Result<(), SynthesisError>
	{		
		if let Some(a) = self.a{
			let a_array = self.a.unwrap();
			let b_array = self.b.unwrap();
			let w_array = self.w.unwrap();
			for i in 0..LEN{
				let mut a_value = Some(a_array[i]);
				let mut b_value = Some(b_array[i]);
				let mut w_value = Some(w_array[i]);
				let mut document = cs.alloc(|| "a",|| a_value.ok_or(SynthesisError::AssignmentMissing))?;
				let mut redactor = cs.alloc(|| "b",|| b_value.ok_or(SynthesisError::AssignmentMissing))?;
				let mut redacted = cs.alloc_input(|| "w",|| w_value.ok_or(SynthesisError::AssignmentMissing))?;
				cs.enforce(
            			|| "mult",
            			|lc| lc + document,
            			|lc| lc + redactor,
            			|lc| lc + redacted
        		  );	
			}
		}else{
		 for i in 0..LEN{			
			let mut a_value = None;
			let mut b_value = None;
			let mut w_value = None;
			let mut document = cs.alloc(|| "a",|| a_value.ok_or(SynthesisError::AssignmentMissing))?;
			let mut redactor = cs.alloc(|| "b",|| b_value.ok_or(SynthesisError::AssignmentMissing))?;
			let mut redacted = cs.alloc_input(|| "w",|| w_value.ok_or(SynthesisError::AssignmentMissing))?;
			cs.enforce(
            			|| "mult",
            			|lc| lc + document,
            			|lc| lc + redactor,
            			|lc| lc + redacted
        		  );
			}
		}
		Ok(())
	}

}
//a[i]*b[i] == c[i]
//p(x)*g(x) == h(x)

provider "aws" {
  profile                 = "default"
  region                  = "eu-central-1"
}

data "template_file" "init_jenkins" {
  template = "${file("${path.module}/template/jenkins.tpl")}"
}

data "template_file" "init_docker" {
  template = "${file("${path.module}/template/docker.tpl")}"
}

resource "aws_instance" "jenkins_master" {
  ami             = "ami-0bdf93799014acdc4"
  instance_type   = "t2.micro"
  key_name        = "virtualBox"
  user_data       = "${data.template_file.init_jenkins.rendered}"
  security_groups = ["${aws_security_group.sg1.name}"]
    tags = {Name ="jenkins_master"}
}

resource "aws_instance" "jenkins_slave" {
  ami             = "ami-0bdf93799014acdc4"
  instance_type   = "t2.micro"
  key_name        = "virtualBox"
  user_data       = "${data.template_file.init_docker.rendered}"
  security_groups = ["${aws_security_group.sg1.name}"]
    tags ={Name= "jenkins_slave"}
}

resource "aws_security_group" "sg1" {
  name        = "sg_allow_ssh"
  description = "Allow ssh 22 port"
  vpc_id      = "vpc-87e2e4ec"

  ingress {
    # TLS (change to whatever ports you need)
    from_port = 22
    to_port   = 22
    protocol  = "tcp"

    # Please restrict your ingress to only necessary IPs and ports.
    # Opening to 0.0.0.0/0 can lead to security vulnerabilities.
    cidr_blocks = ["0.0.0.0/0"] # add a CIDR block here
  }

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }

  ingress {
    from_port   = 8080
    to_port     = 8080
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }
}
